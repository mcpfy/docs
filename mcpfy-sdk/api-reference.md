
# API Reference

This page provides a reference for the public APIs exposed by `mcpfy-sdk`.

The examples in this document target `mcpfy-sdk@0.3.1`.

## Server

Import the server APIs from:

```typescript
import {
  MCPServer,
  object,
  text,
  markdown,
} from "mcpfy-sdk/server";
````

### `MCPServer`

Creates an MCP server.

```typescript
const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});
```

### `MCPServerConfig`

The server configuration supports:

```typescript
interface MCPServerConfig {
  name: string;
  version: string;
  description?: string;
  basePath?: string;
  icon?: string | ServerIcon;
  auth?: AuthConfig;
  widgetsDir?: string;
}
```

#### `name`

Required server name.

```typescript
name: "weather-server"
```

#### `version`

Required server version.

```typescript
version: "1.0.0"
```

#### `description`

Optional description for the server.

```typescript
description: "Provides weather information"
```

#### `basePath`

Optional HTTP MCP endpoint path.

The default is:

```text
/mcp
```

Example:

```typescript
basePath: "/weather"
```

#### `icon`

Optional server icon.

mcpfy supports remote URLs, data URIs, local file paths, and `file:` URLs.

#### `auth`

Optional HTTP authentication configuration.

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

See [Authentication](authentication.md).

#### `widgetsDir`

Optional widget root directory.

The default is:

```text
src/widgets
```

### `server.tool()`

Registers a tool with the MCP server.

```typescript
server.tool(
  {
    name: "add",
    description: "Add two numbers",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  },
  async ({ a, b }) => {
    return object({
      result: a + b,
    });
  }
);
```

The tool definition contains:

* `name`
* `description`
* `schema`
* optional widget configuration

The callback receives validated input.

### Tool Return Helpers

#### `text()`

Creates a text result:

```typescript
return text("Hello!");
```

#### `markdown()`

Creates a Markdown result:

```typescript
return markdown("# Hello\n\nThis is Markdown.");
```

#### `object()`

Creates a structured result:

```typescript
return object({
  success: true,
  value: 42,
});
```

Tool callbacks should use these helpers instead of returning an arbitrary object directly.

## `server.listen()`

Starts the MCP server.

### stdio

```typescript
await server.listen({
  transport: "stdio",
});
```

### HTTP

```typescript
const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(result.url);
```

HTTP listen results include information such as:

```typescript
{
  transport: "http",
  port: 4000,
  host: "localhost",
  url: "http://localhost:4000/mcp"
}
```

### Port Resolution

The HTTP port is resolved in this order:

1. `listen()` option
2. `--port` command-line argument
3. `PORT` environment variable
4. `3000`

For example:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

takes precedence over:

```bash
node server.js --port 5000
```

and:

```text
PORT=6000
```

### Host

The HTTP host can be configured with:

```typescript
await server.listen({
  transport: "http",
  host: "0.0.0.0",
  port: 4000,
});
```

### Silent Startup

Suppress the HTTP startup message with:

```typescript
await server.listen({
  transport: "http",
  silent: true,
});
```

## `server.close()`

Closes the server and its underlying resources.

```typescript
await server.close();
```

## `server.nativeServer`

Provides access to the underlying native MCP server.

```typescript
const nativeServer = server.nativeServer;
```

This can be used when functionality from the official MCP SDK is required directly.

---

# Client

Import the client from:

```typescript
import { MCPClient } from "mcpfy-sdk/client";
```

## `MCPClient`

Creates an MCP client.

### HTTP Server

```typescript
const client = new MCPClient({
  mcpServers: {
    remote: { url: "http://localhost:4000/mcp" },
  },
});

const session = await client.createSession("remote");
```

### stdio Server

```typescript
const client = new MCPClient({
  mcpServers: {
    local: { command: "node", args: ["dist/server.js"] },
  },
});

const session = await client.createSession("local");
```

The client determines the connection type from the server configuration.

A `transport` property is not required for stdio configuration.

---

# Resources

## `server.resource()`

Registers a resource.

```typescript
server.resource(
  {
    name: "server-info",
    uri: "info://server",
    description: "Server information",
    mimeType: "text/plain",
  },
  async () => ({
    contents: [
      {
        uri: "info://server",
        text: "Server information",
      },
    ],
  })
);
```

The callback receives the tool context. The registered URI is available from the definition.

## `server.resourceTemplate()`

Registers a resource template.

The callback signature receives:

```typescript
(uri, params, ctx)
```

Example:

```typescript
server.resourceTemplate(
  {
    name: "user-profile",
    uriTemplate: "user://{id}",
    description: "User profile",
    mimeType: "text/plain",
  },
  async (uri, params, ctx) => ({
    contents: [
      {
        uri: uri.href,
        text: `User: ${params.id}`,
      },
    ],
  })
);
```

The parameters are:

* `uri` — resolved resource URI
* `params` — values extracted from the template
* `ctx` — request context

---

# Prompts

Prompts are reusable prompt definitions exposed by an MCP server.

## Prompt Definition

Prompt arguments are described using a Zod schema.

```typescript
server.prompt(
  {
    name: "greeting",
    description: "Create a greeting",
    schema: z.object({
      name: z.string(),
    }),
  },
  async ({ name }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Greet ${name}`,
          },
        },
      ],
    };
  }
);
```

The prompt definition uses `schema`; there is no `arguments` property on the prompt definition.

---

# Context

Tool and resource callbacks can receive a context object.

## `ctx.log()`

Logs a message using a specified level.

```typescript
ctx.log("info", "Processing request");
```

The first argument is the log level and the second is the message.

---

# Authentication

Import the authentication helpers from the server package:

```typescript
import {
  jwksVerifier,
} from "mcpfy-sdk/server";
```

## `jwksVerifier()`

Creates a JWT verifier using a JWKS endpoint.

```typescript
const verifyToken = jwksVerifier({
  issuer: "https://auth.example.com",
  jwksUri: "https://auth.example.com/.well-known/jwks.json",
  audience: "my-mcp-server",
});
```

Use it in the server configuration:

```typescript
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
```

The authentication discriminant is:

```typescript
type: "oauth"
```

not:

```typescript
type: "jwt"
```

## `NodeOAuthClientProvider`

For Node.js OAuth clients, create the provider through its static `create()` method:

```typescript
const provider = await NodeOAuthClientProvider.create({
  // provider options
});
```

The constructor is private and should not be called directly.

## `ensureAuthorized()`

Ensures that an OAuth provider is authorized for a server.

```typescript
await ensureAuthorized(
  provider,
  serverUrl
);
```

Both the provider and the target server URL are required.

Example:

```typescript
const serverUrl = "https://example.com/mcp";

await ensureAuthorized(
  provider,
  serverUrl
);
```

---

# Authentication Header Forwarding

mcpfy exposes helpers for forwarding supported authentication headers to upstream services.

## `extractForwardableAuthHeaders`

Extracts supported authentication headers from incoming headers.

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

## `forwardAuthHeaders`

Prepares supported authentication headers for forwarding:

```typescript
import type { ToolContext } from "mcpfy-sdk/server";

async function fetchWithContext(ctx: Pick<ToolContext, "requestHeaders" | "auth">) {
  return fetch("https://api.example.com/data", {
    headers: forwardAuthHeaders(ctx),
  });
}
```

## `FORWARDABLE_AUTH_HEADER_NAMES`

The SDK exports:

```typescript
FORWARDABLE_AUTH_HEADER_NAMES
```

This identifies the authentication-related header names that can be forwarded.

Applications should not blindly forward arbitrary inbound headers.

---

# Widgets

Widgets can be associated with tools.

```typescript
server.tool(
  {
    name: "weather",
    description: "Get weather information",
    schema: z.object({
      city: z.string(),
    }),
    widget: "weather",
  },
  async ({ city }) => {
    return object({
      city,
      temperature: 24,
    });
  }
);
```

The widget name corresponds to the widget directory:

```text
src/widgets/weather/
```

The standard entry point is:

```text
src/widgets/weather/main.tsx
```

Build widgets using:

```bash
mcpfy build
```

For development:

```bash
mcpfy dev
```

See [Widgets](widgets.md).

---

# Widget Content

Widget content can be represented as HTML:

```typescript
{
  type: "html",
  html: "<div>Hello</div>",
}
```

or as a URL:

```typescript
{
  type: "url",
  url: "https://example.com/widget",
}
```

HTML should not be supplied as a bare string.

## Widget Size

Widget dimensions use a width/height tuple:

```typescript
size: ["800px", "600px"]
```

---

# Widget React APIs

The React integration is available through the widget React package.

Common APIs include:

* `useHostProtocol()`
* `useCallTool()`
* `useToolPayload()`
* `CallToolHandle`

For detailed React integration, see [Widget React](widget-react.md).

---

# Widget Bridge

The widget bridge provides communication between widget applications and the MCP host.

The package exposes APIs including:

```typescript
postIntent
postNotify
postToolCall
postPrompt
postLink
connectMcpApps
App
PostMessageTransport
getOpenAiGlobal
mcpUiActions
```

These APIs allow widget applications to communicate with the host runtime.

---

# CLI

mcpfy provides CLI commands for widget development and production builds.

## mcpfy dev

Starts the development workflow:

```bash
mcpfy dev
```

## `mcpfy build`

Builds widget assets for production:

```bash
mcpfy build
```

Production deployments should run the build before starting a server that depends on built widgets.

---

# `create-mcpfy-app`

For a new project, the recommended fast-start command is:

```bash
npx create-mcpfy-app@latest
```

This creates a new mcpfy application using the project scaffolding provided by the SDK ecosystem.

---

# Complete Example

The following example combines the core APIs:

```typescript
import {
  MCPServer,
  object,
  text,
} from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
  description: "A calculator MCP server",
});

server.tool(
  {
    name: "add",
    description: "Add two numbers",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  },
  async ({ a, b }) => {
    return object({
      result: a + b,
    });
  }
);

server.tool(
  {
    name: "greet",
    description: "Greet a person",
    schema: z.object({
      name: z.string(),
    }),
  },
  async ({ name }) => {
    return text(`Hello, ${name}!`);
  }
);

server.resource(
  {
    name: "server-info",
    uri: "info://server",
    description: "Server information",
    mimeType: "text/plain",
  },
  async () => ({
    contents: [
      {
        uri: "info://server",
        text: "Calculator MCP server",
      },
    ],
  })
);

await server.listen({
  transport: "http",
  port: 4000,
});
```

The server exposes:

* two executable tools
* one MCP resource
* an HTTP MCP endpoint
* structured and text tool responses

For complete guides and usage examples, see the individual documentation pages for servers, clients, tools, resources, prompts, widgets, and authentication.

