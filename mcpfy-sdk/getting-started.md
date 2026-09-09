# Getting Started

This guide shows how to install `mcpfy-sdk`, create an MCP server, register tools, run the server over stdio or HTTP, and connect to MCP servers using the mcpfy client.

mcpfy is a lightweight TypeScript SDK built on top of the official Model Context Protocol SDK. It provides higher-level APIs for common MCP operations while keeping access to the underlying official MCP implementation when needed.

---

## Prerequisites

Before using mcpfy, make sure your development environment has:

* Node.js `^20.19.0` or `>=22.12.0` (Node.js 21.x is not supported)
* npm, pnpm, or another Node.js package manager
* Basic TypeScript knowledge
* Basic understanding of MCP concepts such as servers, clients, tools, resources, and prompts

The package declares the following Node.js engine requirement:

```text
^20.19.0 || >=22.12.0
```

---

## Installation

Install the SDK using your preferred package manager.

### npm

```bash
npm install mcpfy-sdk
```

### pnpm

```bash
pnpm add mcpfy-sdk
```

### yarn

```bash
yarn add mcpfy-sdk
```

mcpfy uses ES modules, so projects consuming the package should use an ESM-compatible configuration.

For example:

```json
{
  "type": "module"
}
```

---

## Creating a TypeScript Project

A simple MCP server project can be structured as:

```text
my-mcp-server/
├── package.json
├── tsconfig.json
└── src/
    └── server.ts
```

A minimal `package.json` can look like:

```json
{
  "name": "my-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "tsx src/server.ts"
  },
  "dependencies": {
    "mcpfy-sdk": "^0.3.1",
    "zod": "^3.25.0"
  },
  "devDependencies": {
    "tsx": "^4.19.2",
    "typescript": "^5.6.3"
  }
}
```

---

## Creating Your First MCP Server

The main server class is `MCPServer`.

Import it from:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
```

Create a server by providing a name and version:

```typescript
const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});
```

`MCPServer` wraps the official MCP `McpServer` implementation and provides higher-level APIs for registering tools, prompts, resources, resource templates, and widgets.

---

## Adding a Tool

Tools are executable functionality exposed by an MCP server.

Register a tool with:

```typescript
server.tool(definition, callback?);
```

For example:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

server.tool(
  {
    name: "add",
    description: "Add two numbers",
  },
  async ({ a, b }) => {
    return object({
      result: a + b,
    });
  }
);

await server.listen();
```

The tool definition can include a Zod input schema:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
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
  async ({ a, b }) => object({
    result: a + b,
  })
);

await server.listen();
```

A tool callback receives the tool input and the mcpfy `ToolContext`:

```typescript
async (params, ctx) => {
  // ...
}
```

The callback returns a tool result or a supported mcpfy content result.

Tools can also define an `outputSchema` when structured output should be constrained.

---

## Starting the Server

mcpfy supports two transport modes:

* `stdio`
* `http`

The default transport is `stdio`.

Therefore:

```typescript
await server.listen();
```

is equivalent to:

```typescript
await server.listen({
  transport: "stdio",
});
```

---

## Using the stdio Transport

stdio is commonly used when an MCP host launches and manages the server process itself.

Example:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

server.tool(
  {
    name: "add",
    description: "Add two numbers",
  },
  async ({ a, b }) => object({
    result: a + b,
  })
);

await server.listen({
  transport: "stdio",
});
```

The server communicates through standard input and standard output.

The stdio `listen()` result is:

```json
{
  "transport": "stdio"
}
```

---

## Using the HTTP Transport

mcpfy can also expose an MCP server over HTTP.

Use:

```typescript
await server.listen({
  transport: "http",
});
```

Example:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

server.tool(
  {
    name: "add",
    description: "Add two numbers",
  },
  async ({ a, b }) => object({
    result: a + b,
  })
);

const result = await server.listen({
  transport: "http",
});

console.log(result.url);
```

By default, the HTTP server uses:

```text
Host: localhost
Port: 3000
Path: /mcp
```

Therefore, the default endpoint is:

```text
http://localhost:3000/mcp
```

---

## Configuring the HTTP Port

The HTTP port can be configured directly:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

The server will then be available at:

```text
http://localhost:4000/mcp
```

mcpfy resolves the HTTP port using this priority:

1. `listen()` `port` option
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

takes priority over:

```bash
node server.js --port 5000
```

and:

```text
PORT=6000
```

You can also pass `0` as the port to let the operating system choose a free port. The actual bound port is returned by `listen()`.

---

## Command-Line Port Configuration

mcpfy supports both:

```text
--port 4000
```

and:

```text
--port=4000
```

For example:

```bash
node server.js --port 8080
```

The exported `parsePortFromArgv()` function reads these arguments:

```typescript
import { parsePortFromArgv } from "mcpfy-sdk/server";

const port = parsePortFromArgv();
```

If multiple valid `--port` arguments are present, the last one wins.

---

## Using the `PORT` Environment Variable

The port can also be supplied through the environment.

On PowerShell:

```powershell
$env:PORT=5000
npm run dev
```

On Unix-like shells:

```bash
PORT=5000 npm run dev
```

The environment variable is used when no explicit `listen()` port or valid command-line port is provided.

---

## Using a Custom MCP Path

The default MCP HTTP pathname is:

```text
/mcp
```

A custom path can be configured through `basePath`:

```typescript
const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
  basePath: "/weather",
});

await server.listen({
  transport: "http",
});
```

The MCP endpoint will then be:

```text
http://localhost:3000/weather
```

---

## Using a Custom Host

The HTTP host can be configured using the `host` option:

```typescript
await server.listen({
  transport: "http",
  host: "0.0.0.0",
  port: 4000,
});
```

The default host is:

```text
localhost
```

The `host` option is only relevant to HTTP transport.

---

## Suppressing the HTTP Startup Message

By default, mcpfy logs the local MCP URL when an HTTP server starts.

This can be disabled using:

```typescript
await server.listen({
  transport: "http",
  silent: true,
});
```

---

## Understanding the Listen Result

`listen()` returns a `ListenResult`.

For HTTP:

```typescript
const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(result);
```

The result contains:

```json
{
  "transport": "http",
  "port": 4000,
  "host": "localhost",
  "url": "http://localhost:4000/mcp"
}
```

The HTTP `port`, `host`, and `url` describe the bound HTTP server.

For stdio:

```typescript
const result = await server.listen({
  transport: "stdio",
});
```

the result is:

```json
{
  "transport": "stdio"
}
```

---

## Complete HTTP Server Example

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
  description: "A simple calculator MCP server",
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
  async ({ a, b }) => object({
    result: a + b,
  })
);

const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(`MCP server running at ${result.url}`);
```

---

## Server Metadata

When creating an `MCPServer`, the configuration is:

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

### `name`

Required server name.

```typescript
name: "weather-server"
```

### `version`

Required server version.

```typescript
version: "1.0.0"
```

### `description`

Optional server description.

```typescript
description: "Provides weather information"
```

The description is passed to the underlying MCP server as its instructions.

### `basePath`

Optional HTTP pathname for the MCP endpoint.

Default:

```text
/mcp
```

### `icon`

Optional server icon advertised through MCP initialization.

Supported sources include:

* Remote URLs
* `data:` URIs
* Local file paths
* `file:` URLs

For example:

```typescript
const server = new MCPServer({
  name: "weather",
  version: "1.0.0",
  icon: "./icon.png",
});
```

Local icons are converted to data URIs before being advertised.

### `auth`

Optional HTTP authentication configuration.

```typescript
const server = new MCPServer({
  name: "protected-server",
  version: "1.0.0",
  auth: verifier,
});
```

Authentication applies to HTTP transport.

Authentication helpers such as `jwksVerifier`, `oauthAuth0Provider`, and `oauthWorkOSProvider` are available from the server package.

### `widgetsDir`

Optional root directory for widget folders.

Default:

```text
src/widgets
```

This directory is used when tools reference widgets by folder name.

---

## Accessing the Native MCP Server

mcpfy does not completely hide the official MCP SDK.

The underlying official server instance is available through:

```typescript
server.nativeServer
```

For example:

```typescript
const nativeServer = server.nativeServer;
```

Its type is based on the official MCP SDK's `McpServer`.

This provides an escape hatch for advanced use cases that require direct access to the underlying MCP implementation.

---

## Closing a Server

A server can be stopped using:

```typescript
await server.close();
```

Example:

```typescript
const server = new MCPServer({
  name: "example",
  version: "1.0.0",
});

await server.listen({
  transport: "http",
  port: 4000,
});

// Later
await server.close();
```

`close()`:

* Closes the HTTP server if it is running.
* Closes mounted remote connections.
* Clears the remote connection list.
* Closes the underlying native MCP server.

---

## Refreshing MCP Data

mcpfy provides methods for notifying clients when registered MCP data changes.

### Refresh a resource

```typescript
await server.refreshResource("resource://example");
```

This notifies subscribed clients that the resource has new content.

### Refresh resources

```typescript
server.refreshResources();
```

This tells clients to refresh their resource list.

### Refresh tools

```typescript
server.refreshTools();
```

This tells clients to refresh their tool list.

### Refresh prompts

```typescript
server.refreshPrompts();
```

This tells clients to refresh their prompt list.

Resource subscriptions and resource-list-change notifications are enabled by the server runtime.

---

## Using the MCP Client

mcpfy also provides a client abstraction for connecting to configured MCP servers.

Import it from:

```typescript
import { MCPClient } from "mcpfy-sdk/client";
```

Create a client with an `mcpServers` configuration:

```typescript
const client = new MCPClient({
  mcpServers: {
    weather: {
      // server configuration
    },
  },
});
```

The client maintains MCP sessions for configured servers.

### Creating a Session

Create a session for a configured server:

```typescript
const session = await client.createSession("weather");
```

If a session for that server already exists, mcpfy reuses it rather than creating another connection.

### Creating All Sessions

To create sessions for all configured servers:

```typescript
const sessions = await client.createAllSessions();
```

The result is keyed by the configured server names:

```typescript
{
  weather: session,
}
```

---

## Project Structure Recommendation

A small MCP project can use:

```text
my-mcp-server/
├── package.json
├── tsconfig.json
└── src/
    ├── server.ts
    └── tools/
        └── calculator.ts
```

For a larger project:

```text
my-mcp-server/
├── package.json
├── tsconfig.json
└── src/
    ├── server.ts
    ├── tools/
    │   ├── calculator.ts
    │   └── weather.ts
    ├── prompts/
    │   └── assistant.ts
    ├── resources/
    │   └── documentation.ts
    └── widgets/
        └── weather/
            └── ...
```

This structure is not required by the SDK, but provides a clean way to organize larger MCP applications.

---

## Package Entry Points

mcpfy exposes functionality through separate package entry points.

### Main package

```typescript
import ... from "mcpfy-sdk";
```

### Server

```typescript
import {
  MCPServer,
  parsePortFromArgv,
} from "mcpfy-sdk/server";
```

The server entry point includes:

* `MCPServer`
* `parsePortFromArgv`
* Server configuration and listen types
* Tool and prompt types
* Resource types
* Tool context types
* Widget types
* Authentication helpers
* Response helpers
* Server icon types
* Remote server configuration

### Client

```typescript
import {
  MCPClient,
  MCPSession,
  HttpConnector,
  StdioConnector,
} from "mcpfy-sdk/client";
```

The client entry point provides:

* `MCPClient`
* `MCPSession`
* `BaseConnector`
* `StdioConnector`
* `HttpConnector`
* `createConnectorFromConfig`

### React Widget

React widget functionality is exposed through:

```typescript
import {
  HostRuntime,
  useCallTool,
  useHostContext,
  useToolPayload,
} from "mcpfy-sdk/widget";
```

See the widget documentation for the complete React widget API.

---

## API Summary

### `MCPServer`

Creates and manages an MCP server.

```typescript
const server = new MCPServer({
  name: "example",
  version: "1.0.0",
});
```

### `server.tool()`

Registers an MCP tool.

```typescript
server.tool(definition, callback?);
```

### `server.prompt()`

Registers an MCP prompt.

```typescript
server.prompt(definition, callback?);
```

### `server.resource()`

Registers a static MCP resource.

```typescript
server.resource(definition, callback?);
```

### `server.resourceTemplate()`

Registers a dynamic MCP resource template.

```typescript
server.resourceTemplate(definition, callback?);
```

### `server.listen()`

Starts the MCP server.

```typescript
await server.listen({
  transport: "stdio",
});
```

or:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

### `server.close()`

Stops the server and closes associated connections.

```typescript
await server.close();
```

### `server.nativeServer`

Provides direct access to the underlying official MCP server.

```typescript
server.nativeServer;
```

### `server.refreshResource()`

Notifies subscribed clients that a resource has changed.

```typescript
await server.refreshResource("resource://example");
```

### `server.refreshResources()`

Notifies clients that the resource list should be refreshed.

```typescript
server.refreshResources();
```

### `server.refreshTools()`

Notifies clients that the tool list should be refreshed.

```typescript
server.refreshTools();
```

### `server.refreshPrompts()`

Notifies clients that the prompt list should be refreshed.

```typescript
server.refreshPrompts();
```

### `parsePortFromArgv()`

Reads a valid `--port` argument from command-line arguments.

```typescript
const port = parsePortFromArgv();
```

The last valid port argument wins.

---

## Best Practices

### Use `MCPServer` for common server operations

Use the higher-level mcpfy APIs rather than manually registering every operation against the native MCP server.

```typescript
server.tool(...);
server.prompt(...);
server.resource(...);
server.resourceTemplate(...);
```

### Use schemas for tool inputs

When a tool expects structured input, define its input schema explicitly:

```typescript
schema: z.object({
  city: z.string(),
})
```

This makes the expected tool input clear and allows the underlying MCP registration to use the schema.

### Use HTTP and stdio according to the deployment environment

Use stdio when an MCP host launches the server as a child process.

Use HTTP when the MCP server needs to be exposed as an HTTP endpoint.

### Keep server metadata meaningful

Use descriptive names, versions, descriptions, and paths:

```typescript
const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
  description: "Provides weather information",
  basePath: "/weather",
});
```

### Use the native server only when necessary

For common operations, prefer mcpfy's higher-level APIs.

Use:

```typescript
server.nativeServer
```

when direct access to functionality from the official MCP SDK is required.

### Detect optional widget capabilities

Widgets should not assume every MCP host supports every optional feature. Use the widget runtime's capability information before relying on host-specific functionality.

---

## Summary

mcpfy provides a lightweight API for building MCP applications while remaining compatible with the official MCP SDK.

The main server workflow is:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "example-server",
  version: "1.0.0",
});

server.tool(
  {
    name: "hello",
    description: "Return a greeting",
  },
  async ({ name }) => object({
    message: `Hello, ${name}!`,
  })
);

await server.listen({
  transport: "stdio",
});
```

For HTTP:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

For clients:

```typescript
import { MCPClient } from "mcpfy-sdk/client";

const client = new MCPClient({
  mcpServers: {
    weather: {
      // server configuration
    },
  },
});

const session = await client.createSession("weather");
```

mcpfy provides higher-level APIs for servers, clients, tools, prompts, resources, resource templates, authentication, and widgets while retaining access to the underlying official MCP implementation when advanced control is required.
