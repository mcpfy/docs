# Authentication

mcpfy provides authentication support for HTTP-based MCP servers using bearer tokens. Authentication is optional and can be configured when creating an `MCPServer`.

The authentication layer is designed to work with standard OAuth/OIDC and JWT-based authorization systems while keeping the authentication logic separate from the MCP server implementation.

## Overview

mcpfy supports two authentication approaches:

1. **Custom header verification** — validate a bearer token using a custom verification function.
2. **JWT/JWKS verification** — validate standard JWT access tokens using an issuer, JWKS endpoint, and optional audience.

Authentication is supported for **HTTP transport only**. Stdio-based MCP servers do not use the HTTP bearer-token authentication middleware.

The overall flow is:

```text
Client
  │
  │ Authorization: Bearer <token>
  ▼
mcpfy HTTP Server
  │
  ▼
Bearer Token Extraction
  │
  ▼
Configured Auth Verifier
  │
  ├── Header verifier
  │
  └── JWT/JWKS verifier
  │
  ▼
Authentication Result
  │
  ├── Valid → MCP request continues
  │
  └── Invalid → Request is rejected
```

---

## Configuring Authentication

Authentication is configured through the `auth` property of `MCPServerConfig`.

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "authenticated-server",
  version: "1.0.0",
  auth: {
    // authentication configuration
  },
});
```

The authentication configuration is passed to the HTTP transport when the server starts.

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});
```

---

# Bearer Token Authentication

mcpfy expects the client to provide the access token using the standard HTTP `Authorization` header:

```http
Authorization: Bearer <access-token>
```

The authentication middleware:

1. Reads the `Authorization` header.
2. Checks that the authentication scheme is `Bearer`.
3. Extracts the token.
4. Passes the token to the configured verifier.
5. Continues the request only when verification succeeds.

For example:

```http
GET /mcp HTTP/1.1
Host: localhost:3000
Authorization: Bearer eyJhbGciOi...
```

Requests without a valid bearer token are rejected.

---

# Authentication Configuration Types

mcpfy exposes the `AuthConfig` and `AuthInfo` types from the server package.

```typescript
import type { AuthConfig, AuthInfo } from "mcpfy-sdk/server";
```

`AuthInfo` represents information associated with an authenticated request.

A verified authentication result can contain information such as:

```typescript
{
  sub: "user-id",
  scopes: ["read", "write"],
  claims: {
    // JWT claims
  },
  token: "access-token"
}
```

The exact claims depend on the authorization server and token being verified.

---

# JWT/JWKS Authentication

For standard OAuth/OIDC authorization servers that issue signed JWT access tokens, mcpfy provides the `jwksVerifier` helper.

```typescript
import { MCPServer, jwksVerifier } from "mcpfy-sdk/server";
```

The verifier uses:

* `issuer`
* `jwksUri`
* optional `audience`

to validate the JWT.

## Basic Example

```typescript
import { MCPServer, jwksVerifier } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "secure-server",
  version: "1.0.0",

  auth: {
    type: "jwt",
    verifyToken: jwksVerifier({
      issuer: "https://issuer.example.com",
      jwksUri: "https://issuer.example.com/.well-known/jwks.json",
      audience: "my-mcp-api",
    }),
  },
});

await server.listen({
  transport: "http",
  port: 3000,
});
```

The `jwksVerifier` function creates a reusable verifier that retrieves the authorization server's JSON Web Key Set and uses it to verify incoming JWT signatures.

---

# JwksVerifierOptions

The `jwksVerifier` function accepts the following configuration:

```typescript
interface JwksVerifierOptions {
  issuer: string;
  jwksUri: string;
  audience?: string;
}
```

## `issuer`

The expected JWT `iss` claim.

```typescript
issuer: "https://issuer.example.com"
```

The token must contain a matching issuer.

---

## `jwksUri`

The URL of the authorization server's JWKS endpoint.

```typescript
jwksUri: "https://issuer.example.com/.well-known/jwks.json"
```

The JWKS endpoint provides the public keys required to verify signed JWTs.

---

## `audience`

An optional expected JWT `aud` claim.

```typescript
audience: "my-mcp-api"
```

If supplied, the JWT must contain the expected audience.

---

# How JWT Verification Works

Internally, mcpfy uses the `jose` library to perform JWT verification.

The verifier creates a remote JWKS resolver:

```typescript
const jwks = createRemoteJWKSet(new URL(options.jwksUri));
```

For each token, the verifier performs JWT validation using:

```typescript
jwtVerify(token, jwks, {
  issuer: options.issuer,
  audience: options.audience,
});
```

A successfully verified token is converted into `AuthInfo`.

The `sub` claim is exposed as the authenticated subject:

```typescript
{
  sub: payload.sub
}
```

The `scope` claim is also processed when present.

For example, a JWT containing:

```json
{
  "sub": "user-123",
  "scope": "read write"
}
```

produces:

```typescript
{
  sub: "user-123",
  scopes: ["read", "write"]
}
```

All JWT claims are also available through the `claims` property.

---

# Scope Handling

mcpfy recognizes the standard space-separated `scope` claim.

For example:

```json
{
  "scope": "read write admin"
}
```

is converted to:

```typescript
scopes: ["read", "write", "admin"]
```

If the token does not contain a string `scope` claim, `scopes` is left undefined.

mcpfy does not automatically enforce individual scopes. Applications can use the authenticated information to implement their own authorization rules.

For example:

```typescript
if (!ctx.auth?.scopes?.includes("admin")) {
  throw new Error("Admin scope required");
}
```

---

# Custom Header Authentication

mcpfy also supports a simpler custom verification mechanism.

This is useful when an application already has its own token validation system or does not need JWT/JWKS verification.

Example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "custom-auth-server",
  version: "1.0.0",

  auth: {
    type: "header",
    verify: async (token) => {
      return token === process.env.MCP_TOKEN;
    },
  },
});

await server.listen({
  transport: "http",
  port: 3000,
});
```

The `verify` function receives the bearer token:

```typescript
verify: async (token) => {
  // Validate token
  return true;
}
```

It must return:

* `true` when the token is valid
* `false` when the token is invalid

For a valid custom header token, mcpfy creates an authentication result containing the token and an empty claims object.

---

# Authentication Middleware

The authentication middleware is implemented in:

```text
src/server/auth/middleware.ts
```

The main function is:

```typescript
checkAuth(
  req: IncomingMessage,
  config: AuthConfig
): Promise<AuthCheckResult>
```

Its responsibility is intentionally small: extract the bearer token and delegate verification to the configured authentication strategy.

The bearer token is extracted from:

```http
Authorization: Bearer <token>
```

The middleware rejects the request when:

* the `Authorization` header is missing
* the scheme is not `Bearer`
* the token is missing
* the configured verifier rejects the token
* JWT verification fails

---

# Authentication Result

The authentication middleware returns one of two results:

```typescript
type AuthCheckResult =
  | {
      ok: true;
      auth: AuthInfo;
    }
  | {
      ok: false;
    };
```

A successful result has:

```typescript
{
  ok: true,
  auth: {
    // authentication information
  }
}
```

An unsuccessful result has:

```typescript
{
  ok: false
}
```

This makes authentication checks straightforward for the HTTP transport.

---

# OAuth Providers

mcpfy also exposes OAuth-related helpers from the server package.

```typescript
import {
  oauthAuth0Provider,
  oauthWorkOSProvider,
} from "mcpfy-sdk/server";
```

These helpers provide provider-specific authentication configuration for supported OAuth systems.

The package also exposes:

```typescript
import {
  NodeOAuthClientProvider,
  ensureAuthorized,
  OAuthSessionStore,
  FileKVStore,
} from "mcpfy-sdk/auth";
```

These utilities are intended for OAuth client-side/session management.

---

# OAuth Session Storage

mcpfy provides an `OAuthSessionStore` abstraction for maintaining OAuth session information.

```typescript
import { OAuthSessionStore } from "mcpfy-sdk/auth";
```

For filesystem-based persistence, mcpfy also provides:

```typescript
import { FileKVStore } from "mcpfy-sdk/auth";
```

`FileKVStore` implements the `KVStore` abstraction and can be used where simple persistent key-value storage is required.

The storage abstraction keeps OAuth state management separate from the rest of the MCP server implementation.

---

# Auth0 and WorkOS

mcpfy provides preset helpers for commonly used OAuth providers:

```typescript
oauthAuth0Provider(...)
oauthWorkOSProvider(...)
```

These presets are exported from:

```text
src/server/auth/presets.ts
```

They are intended to reduce provider-specific configuration while still using the same authentication architecture.

For providers not covered by a preset, the generic `jwksVerifier` can be used when the provider exposes a standard JWT issuer and JWKS endpoint.

---

# Authentication with HTTP Transport

Authentication is applied when using HTTP transport:

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});
```

A typical authenticated server therefore looks like:

```typescript
import { MCPServer, jwksVerifier } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "my-secure-mcp-server",
  version: "1.0.0",

  auth: {
    type: "jwt",
    verifyToken: jwksVerifier({
      issuer: process.env.OAUTH_ISSUER!,
      jwksUri: process.env.OAUTH_JWKS_URI!,
      audience: process.env.OAUTH_AUDIENCE,
    }),
  },
});

server.tool(
  {
    name: "private-data",
    description: "Returns private data",
  },
  async () => {
    return {
      data: "Authenticated response",
    };
  }
);

await server.listen({
  transport: "http",
  port: 3000,
});
```

The client must then include a valid bearer token when connecting to the server.

---

# Authentication and Stdio

Authentication configuration is intended for HTTP transport.

For stdio:

```typescript
await server.listen({
  transport: "stdio",
});
```

communication occurs through the local process's standard input/output streams rather than HTTP requests, so HTTP bearer-token authentication is not applied.

This is consistent with common local MCP host usage such as launching an MCP server as a subprocess.

---

# Security Recommendations

## Use HTTPS in Production

Bearer tokens must be protected in transit.

For production deployments, run the MCP HTTP endpoint behind HTTPS or a trusted TLS-terminating reverse proxy.

---

## Do Not Hard-Code Secrets

Avoid:

```typescript
verify: async (token) => token === "my-secret-token"
```

for production applications.

Instead, load secrets from secure environment variables or a dedicated secrets-management system.

Example:

```typescript
const expectedToken = process.env.MCP_TOKEN;

const server = new MCPServer({
  name: "secure-server",
  version: "1.0.0",

  auth: {
    type: "header",
    verify: async (token) => token === expectedToken,
  },
});
```

---

## Validate the Issuer and Audience

When using JWT authentication, configure the expected issuer and, when applicable, audience.

```typescript
jwksVerifier({
  issuer: "https://issuer.example.com",
  jwksUri: "https://issuer.example.com/.well-known/jwks.json",
  audience: "my-mcp-api",
});
```

This prevents accepting tokens issued for an unexpected authorization server or API.

---

## Use the Appropriate Authorization Logic

Authentication answers:

> "Who is making this request?"

Authorization answers:

> "Is this authenticated caller allowed to perform this operation?"

mcpfy handles token verification, but application-specific permission checks should be implemented according to the application's requirements.

For example:

```typescript
if (!auth.scopes?.includes("write")) {
  throw new Error("Insufficient permissions");
}
```

---

# Authentication File Structure

The authentication implementation is organized as follows:

```text
src/
├── auth/
│   ├── file-kv-store.ts
│   ├── index.ts
│   ├── kv-store.ts
│   ├── node-oauth-provider.ts
│   └── oauth-session-store.ts
│
└── server/
    └── auth/
        ├── index.ts
        ├── jwks-verifier.ts
        ├── middleware.ts
        ├── presets.ts
        ├── types.ts
        └── well-known.ts
```

### `server/auth/middleware.ts`

Handles bearer-token extraction and dispatches the token to the configured verifier.

### `server/auth/jwks-verifier.ts`

Provides generic JWT verification using a remote JWKS endpoint.

### `server/auth/presets.ts`

Provides provider-specific OAuth authentication presets.

### `server/auth/types.ts`

Defines authentication configuration and authentication information types.

### `server/auth/well-known.ts`

Contains support for authentication metadata/well-known endpoint functionality.

### `auth/node-oauth-provider.ts`

Provides the Node.js OAuth client provider implementation.

### `auth/oauth-session-store.ts`

Manages OAuth session persistence.

### `auth/file-kv-store.ts`

Provides filesystem-backed key-value storage.

### `auth/kv-store.ts`

Defines the key-value storage abstraction.

---

# API Exports

Authentication functionality is available through the server entry point:

```typescript
import {
  jwksVerifier,
  oauthAuth0Provider,
  oauthWorkOSProvider,
} from "mcpfy-sdk/server";
```

Authentication types:

```typescript
import type {
  AuthConfig,
  AuthInfo,
  JwksVerifierOptions,
} from "mcpfy-sdk/server";
```

OAuth client functionality is available through:

```typescript
import {
  NodeOAuthClientProvider,
  ensureAuthorized,
  OAuthSessionStore,
  FileKVStore,
} from "mcpfy-sdk/auth";
```

---

# Summary

mcpfy provides a framework-independent authentication layer for HTTP MCP servers.

The main authentication flow is:

```text
HTTP Request
     │
     ▼
Authorization Header
     │
     ▼
Bearer Token
     │
     ▼
Authentication Verifier
     │
     ├── Custom Header Verification
     │
     └── JWT/JWKS Verification
             │
             ▼
        Verified Claims
             │
             ▼
       Authenticated MCP Request
```

For standard JWT/OIDC providers, `jwksVerifier` provides a generic solution using the provider's issuer and JWKS endpoint. For applications with custom token systems, the `header` authentication strategy allows the application to provide its own token verification logic.

Authentication is intentionally separated from MCP tool, prompt, resource, and widget implementation so that the same server functionality can be protected without coupling application logic to a particular authentication provider.