
# Getting Started

This guide shows how to install `mcpfy-sdk`, create an MCP server, expose tools, run the server, and consume an MCP server using the mcpfy client.

mcpfy is a lightweight TypeScript SDK built on top of the official Model Context Protocol SDK. It provides a simpler API for building MCP servers and clients while keeping access to the underlying official SDK when required.

---

## Prerequisites

Before using mcpfy, make sure the development environment has:

- Node.js 20.19.0 or newer, or Node.js 22.12.0 or newer
- npm, pnpm, or another Node.js package manager
- Basic TypeScript knowledge
- Basic understanding of MCP concepts such as servers, clients, tools, resources, and prompts

The package declares the following Node.js engine requirement:

```text
^20.19.0 || >=22.12.0
```

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

## Creating a TypeScript Project

A simple project can be structured as:

```
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
    "mcpfy-sdk": "^0.3.1"
  },
  "devDependencies": {
    "tsx": "^4.19.2",
    "typescript": "^5.6.3"
  }
}
```

## Creating Your First MCP Server

The main server class is exported as `MCPServer`.

Import it from:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
```

Create a server by providing a name and version:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});
```

The `MCPServer` class wraps the official MCP server implementation and provides a higher-level API for registering MCP functionality.

### Adding a Tool

Tools are the primary way an MCP server exposes executable functionality to an MCP client.

A tool can be registered using:

```typescript
server.tool(...)
```

For example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

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
    return {
      result: a + b,
    };
  }
);

await server.listen();
```

The exact input schema can also be specified when defining the tool.

For example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
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
  async ({ a, b }) => {
    return {
      result: a + b,
    };
  }
);

await server.listen();
```

The callback receives the validated tool input and returns the tool result.

## Starting the Server

mcpfy supports two transport modes:

- stdio
- http

The default transport is stdio.

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

### Using the stdio Transport

stdio is the default transport and is commonly used when an MCP host launches the server as a child process.

Example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

server.tool(
  {
    name: "add",
    description: "Add two numbers",
  },
  async ({ a, b }) => ({
    result: a + b,
  })
);

await server.listen({
  transport: "stdio",
});
```

The server communicates through standard input and standard output.

This makes stdio suitable for MCP hosts that start and manage the server process themselves.

### Using the HTTP Transport

mcpfy can also expose an MCP server over HTTP.

Use:

```typescript
await server.listen({
  transport: "http",
});
```

Example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

server.tool(
  {
    name: "add",
    description: "Add two numbers",
  },
  async ({ a, b }) => ({
    result: a + b,
  })
);

const result = await server.listen({
  transport: "http",
});

console.log(result.url);
```

By default, the HTTP server uses:

```
Host: localhost
Port: 3000
Path: /mcp
```

Therefore, the default endpoint is:

```
http://localhost:3000/mcp
```

### Configuring the HTTP Port

The HTTP port can be configured directly:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

The server will then be available at:

```
http://localhost:4000/mcp
```

mcpfy resolves the HTTP port using the following priority:

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

takes priority over:

```bash
node server.js --port 5000
```

and over:

```
PORT=6000
```

### Command-Line Port Configuration

mcpfy supports both:

```
--port 4000
```

and:

```
--port=4000
```

For example:

```bash
node server.js --port 8080
```

The SDK parses the port using:

```typescript
parsePortFromArgv()
```

This function returns the last valid port argument when multiple `--port` arguments are supplied.

### Using the PORT Environment Variable

The port can also be supplied through the environment:

```bash
$env:PORT=5000
npm run dev
```

The server will use port 5000 unless a higher-priority configuration is supplied.

### Using a Custom MCP Path

The default MCP HTTP pathname is:

```
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

```
http://localhost:3000/weather
```

### Using a Custom Host

The HTTP host can be configured using the `host` option:

```typescript
await server.listen({
  transport: "http",
  host: "0.0.0.0",
  port: 4000,
});
```

This is useful when the server needs to listen on an interface other than the default localhost.

The default host is:

```
localhost
```

### Suppressing the HTTP Startup Message

By default, mcpfy logs the local HTTP MCP URL when the server starts.

This can be disabled using:

```typescript
await server.listen({
  transport: "http",
  silent: true,
});
```

### Understanding the Listen Result

For HTTP servers, `listen()` returns information about the running server.

Example:

```typescript
const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(result);
```

The result has the following structure:

```json
{
  "transport": "http",
  "port": 4000,
  "host": "localhost",
  "url": "http://localhost:4000/mcp"
}
```

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

### Complete HTTP Server Example

The following is a complete minimal HTTP MCP server:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
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
  async ({ a, b }) => {
    return {
      result: a + b,
    };
  }
);

const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(`MCP server running at ${result.url}`);
```

## Server Metadata

When creating an `MCPServer`, the following configuration is available:

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

### name

Required server name.

```typescript
name: "weather-server"
```

### version

Required server version.

```typescript
version: "1.0.0"
```

### description

Optional server description.

```typescript
description: "Provides weather information"
```

The description is used as part of the server instructions supplied to the underlying MCP implementation.

### basePath

Optional HTTP MCP endpoint pathname.

Default:

```
/mcp
```

### icon

Optional server icon.

mcpfy supports:

- Remote URLs
- data: URIs
- Local file paths
- file: URLs

Local icons can be converted to data URIs so MCP clients can display them.

### auth

Optional authentication configuration.

Authentication applies to HTTP transport.

Authentication configuration and JWT/OIDC verification are covered in the authentication documentation.

### widgetsDir

Optional root directory for widget folders.

The default is:

```
src/widgets
```

## Accessing the Native MCP Server

mcpfy intentionally does not completely hide the official MCP SDK.

The underlying official server instance is available through:

```
server.nativeServer
```

For example:

```typescript
const nativeServer = server.nativeServer;
```

This provides an escape hatch for advanced use cases that require direct access to the underlying MCP SDK.

The main advantage is that developers can use the simpler mcpfy API for common functionality while still retaining access to the official MCP implementation when necessary.

## Closing a Server

An HTTP server can be stopped using:

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

- Closes the HTTP server if it is running.
- Closes mounted remote connections.
- Clears the remote connection list.
- Closes the underlying native MCP server.

## Project Structure Recommendation

A small MCP project can use:

```
my-mcp-server/
├── package.json
├── tsconfig.json
└── src/
    ├── server.ts
    └── tools/
        └── calculator.ts
```

For a larger project:

```
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

This structure is not required by the SDK, but it provides a clean way to organize larger MCP applications.
