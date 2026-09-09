
# Authentication

mcpfy supports authentication for MCP servers exposed over HTTP.

Authentication allows an MCP server to verify incoming access tokens before allowing requests to reach protected MCP endpoints.

## Authentication Configuration

Authentication is configured through the `auth` property of `MCPServer`.

The authentication configuration uses OAuth as the authentication type:

```typescript
import { MCPServer, jwksVerifier } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "protected-server",
  version: "1.0.0",
  auth: {
    type: "oauth",
    verifyToken: jwksVerifier({
      issuer: "https://auth.example.com",
      jwksUri: "https://auth.example.com/.well-known/jwks.json",
      audience: "my-mcp-server",
    }),
    authorizationServers: ["https://auth.example.com"],
  },
});
````

The important properties are:

* `type` — authentication mechanism. For the supported OAuth configuration, use `"oauth"`.
* `verifyToken` — function used to verify incoming access tokens.
* `authorizationServers` — list of authorization server URLs.

`authorizationServers` is required for the OAuth authentication configuration.

## OAuth Authentication

mcpfy represents protected-resource authentication using OAuth metadata.

A typical configuration is:

```typescript
import { MCPServer, jwksVerifier } from "mcpfy-sdk/server";

const issuer = "https://auth.example.com";

const server = new MCPServer({
  name: "protected-server",
  version: "1.0.0",
  auth: {
    type: "oauth",
    verifyToken: jwksVerifier({
      issuer,
      jwksUri: `${issuer}/.well-known/jwks.json`,
      audience: "my-mcp-server",
    }),
    authorizationServers: [issuer],
  },
});

await server.listen({
  transport: "http",
  port: 4000,
});
```

The server can then verify bearer tokens supplied by MCP clients.

## JWT / JWKS Verification

mcpfy provides a JWKS-based verifier for validating JWT access tokens.

A verifier can be created with:

```typescript
import { jwksVerifier } from "mcpfy-sdk/server";

const verifyToken = jwksVerifier({
  issuer: "https://auth.example.com",
  jwksUri: "https://auth.example.com/.well-known/jwks.json",
  audience: "my-mcp-server",
});
```

The verifier uses the issuer, JWKS endpoint, and audience to validate the token.

### Issuer

The `issuer` identifies the authorization server that issued the token.

```typescript
issuer: "https://auth.example.com"
```

### JWKS URI

The `jwksUri` identifies the endpoint containing the public keys used to verify JWT signatures.

```typescript
jwksUri: "https://auth.example.com/.well-known/jwks.json"
```

### Audience

The `audience` identifies the intended recipient of the token.

```typescript
audience: "my-mcp-server"
```

## Complete JWT Verification Example

```typescript
import {
  MCPServer,
  jwksVerifier,
} from "mcpfy-sdk/server";

const issuer = "https://auth.example.com";

const server = new MCPServer({
  name: "protected-server",
  version: "1.0.0",
  auth: {
    type: "oauth",
    verifyToken: jwksVerifier({
      issuer,
      jwksUri: `${issuer}/.well-known/jwks.json`,
      audience: "my-mcp-server",
    }),
    authorizationServers: [issuer],
  },
});

await server.listen({
  transport: "http",
  port: 4000,
});
```

This configuration:

1. Declares the MCP server as OAuth-protected.
2. Configures JWT verification through JWKS.
3. Identifies the authorization server.
4. Exposes the MCP server over HTTP.

## Protected Resource Metadata

When OAuth authentication is configured, mcpfy also exposes protected-resource metadata through the standard:

```text
/.well-known/oauth-protected-resource
```

This endpoint allows MCP clients to discover the authorization server associated with the protected MCP resource.

For example, if the MCP server is available at:

```text
http://localhost:4000/mcp
```

the protected-resource metadata is exposed through the server's well-known endpoint.

This endpoint is activated by the OAuth authentication configuration and should not be treated as a manually created application route.

## Authorization Servers

The authorization server is specified through:

```typescript
authorizationServers: [
  "https://auth.example.com",
]
```

Multiple authorization servers can be supplied when supported by the deployment:

```typescript
authorizationServers: [
  "https://auth.example.com",
  "https://login.example.com",
]
```

The values should identify the actual authorization servers trusted by the MCP server.

## Forwarding Authentication Headers

When an authenticated MCP request needs to make an upstream request, mcpfy provides helpers for forwarding supported authentication headers.

The relevant APIs are:

```typescript
forwardAuthHeaders
extractForwardableAuthHeaders
FORWARDABLE_AUTH_HEADER_NAMES
```

These APIs provide the sanctioned mechanism for forwarding permitted inbound authentication headers rather than manually forwarding arbitrary request headers.

### `extractForwardableAuthHeaders`

Use this helper to extract headers that are allowed to be forwarded.

```typescript
import type { IncomingMessage } from "node:http";
import {
  extractForwardableAuthHeaders,
  forwardAuthHeaders,
} from "mcpfy-sdk/server";

async function fetchUpstream(request: IncomingMessage) {
  const requestHeaders = extractForwardableAuthHeaders(request);

  return fetch("https://api.example.com/data", {
    headers: forwardAuthHeaders({ requestHeaders }),
  });
}
```

`extractForwardableAuthHeaders()` accepts a Node `IncomingMessage` and returns the
allowlisted headers. `forwardAuthHeaders()` accepts a tool-context-shaped object with
`requestHeaders` (or `auth`) and prepares headers for an upstream request.

### `forwardAuthHeaders`

The forwarding helper can be used when making an authenticated upstream request.

```typescript
import type { ToolContext } from "mcpfy-sdk/server";

async function fetchWithContext(ctx: Pick<ToolContext, "requestHeaders" | "auth">) {
  return fetch("https://api.example.com/data", {
    headers: forwardAuthHeaders(ctx),
  });
}
```

Only headers supported by the SDK's forwarding rules should be forwarded.

### `FORWARDABLE_AUTH_HEADER_NAMES`

The SDK also exposes:

```typescript
FORWARDABLE_AUTH_HEADER_NAMES
```

This constant represents the authentication-related header names that the SDK permits for forwarding.

Avoid forwarding arbitrary inbound headers to upstream services.

## Authentication in Tool Context

Authentication can be used together with tool execution.

For example:

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "protected-server",
  version: "1.0.0",
  auth: {
    type: "oauth",
    verifyToken: jwksVerifier({
      issuer: "https://auth.example.com",
      jwksUri: "https://auth.example.com/.well-known/jwks.json",
      audience: "my-mcp-server",
    }),
    authorizationServers: [
      "https://auth.example.com",
    ],
  },
});

server.tool(
  {
    name: "private-data",
    description: "Get private data",
    schema: z.object({}),
  },
  async () => {
    return text("Authenticated request accepted.");
  }
);

await server.listen({
  transport: "http",
  port: 4000,
});
```

The HTTP authentication layer verifies the incoming token before the protected MCP operation is processed.

## Authentication Flow

A typical protected MCP request follows this flow:

```text
MCP Client
    │
    │ Authorization: Bearer <token>
    ▼
MCP HTTP Server
    │
    │ verifyToken()
    ▼
Token Verification
    │
    ├── Invalid → Request rejected
    │
    └── Valid
          │
          ▼
     MCP Request
          │
          ▼
       Tool / Resource
```

The authorization server is responsible for issuing the access token. The MCP server is responsible for validating the token before processing the request.

## Authentication with Custom Authorization Server

The authorization server does not have to use a specific provider. The important requirement is that the server configuration supplies the appropriate issuer, JWKS endpoint, audience, and authorization-server metadata.

For example:

```typescript
const issuer = "https://login.example.com";

const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
  auth: {
    type: "oauth",
    verifyToken: jwksVerifier({
      issuer,
      jwksUri: "https://login.example.com/.well-known/jwks.json",
      audience: "my-mcp-server",
    }),
    authorizationServers: [issuer],
  },
});
```

## Client-Side OAuth Helpers

mcpfy also provides helpers for MCP clients that need to complete an OAuth authorization flow.

### `NodeOAuthClientProvider`

The Node OAuth client provider is created using its static `create()` method.

```typescript
const provider = await NodeOAuthClientProvider.create({
  // provider options
});
```

Do not instantiate it directly with:

```typescript
new NodeOAuthClientProvider(...)
```

The constructor is private.

### `ensureAuthorized`

`ensureAuthorized()` ensures that the OAuth provider is authorized for a specific server URL.

It requires both the provider and the server URL:

```typescript
await ensureAuthorized(
  provider,
  serverUrl
);
```

For example:

```typescript
const serverUrl = "https://example.com/mcp";

await ensureAuthorized(
  provider,
  serverUrl
);
```

## Security Recommendations

### Use HTTPS in production

Authentication tokens should be transmitted over HTTPS in production environments.

Avoid sending bearer tokens over unencrypted HTTP.

### Validate the issuer

Configure the verifier with the expected authorization-server issuer:

```typescript
issuer: "https://auth.example.com"
```

### Validate the audience

Use an audience value appropriate for the MCP server:

```typescript
audience: "my-mcp-server"
```

### Protect signing keys

The MCP server uses the authorization server's public JWKS keys to verify JWT signatures. Private signing keys should remain under the control of the authorization server.

### Do not forward arbitrary headers

When making upstream requests, use mcpfy's authentication-header forwarding helpers instead of copying all inbound headers.

## Troubleshooting

### `type: "jwt"` is rejected

Use:

```typescript
type: "oauth"
```

The authentication configuration uses `"oauth"` as its discriminant.

### `authorizationServers` is missing

OAuth configuration requires:

```typescript
authorizationServers: [
  "https://auth.example.com",
]
```

Make sure the authorization server URL is included.

### Token verification fails

Check:

* `issuer`
* `jwksUri`
* `audience`
* JWT signature
* token expiration
* authorization-server configuration

The issuer and audience configured in the verifier must match the token being presented.

### OAuth metadata is not available

Verify that HTTP authentication is configured using:

```typescript
auth: {
  type: "oauth",
  verifyToken: ...,
  authorizationServers: [...],
}
```

mcpfy uses this configuration to expose the protected-resource metadata endpoint.

## Summary

A basic authenticated MCP server uses:

```typescript
auth: {
  type: "oauth",
  verifyToken: jwksVerifier({
    issuer,
    jwksUri,
    audience,
  }),
  authorizationServers: [issuer],
}
```

The main authentication components are:

* OAuth authentication configuration
* JWT verification
* JWKS public-key discovery
* Authorization-server metadata
* Protected-resource metadata
* Authentication-header forwarding
* OAuth client helpers

For production deployments, use HTTPS, configure the correct issuer and audience, and use the SDK's header-forwarding helpers for authenticated upstream requests.
