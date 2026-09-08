
# API Reference

This page provides a reference for the public APIs exposed by the `mcpfy-sdk` package.

The SDK is organized into separate entry points:

```text
mcpfy-sdk
mcpfy-sdk/server
mcpfy-sdk/client
mcpfy-sdk/widget
mcpfy-sdk/widget-bridge
mcpfy-sdk/auth
```

---

# Main Package

```ts
import { ... } from "mcpfy-sdk";
```

The main package exposes the core SDK functionality and commonly used server APIs.

For server development, prefer the dedicated server entry point:

```ts
import { MCPServer } from "mcpfy-sdk/server";
```

---

# mcpfy-sdk/server

The server entry point provides APIs for creating and running MCP servers.

```ts
import {
  MCPServer,
  text,
  markdown,
  image,
  object,
  error,
} from "mcpfy-sdk/server";
```

## MCPServer

```ts
class MCPServer
```

Creates a high-level MCP server wrapper around the official Model Context Protocol SDK.

### Constructor

```ts
new MCPServer(config: MCPServerConfig)
```

### MCPServerConfig

```ts
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

| Property      | Type                   | Required | Description                                                   |
| ------------- | ---------------------- | -------: | ------------------------------------------------------------- |
| `name`        | `string`               |      Yes | Server name.                                                  |
| `version`     | `string`               |      Yes | Server version.                                               |
| `description` | `string`               |       No | Server description/instructions.                              |
| `basePath`    | `string`               |       No | HTTP MCP endpoint path. Defaults to `/mcp`.                   |
| `icon`        | `string \| ServerIcon` |       No | Server icon configuration.                                    |
| `auth`        | `AuthConfig`           |       No | HTTP authentication configuration.                            |
| `widgetsDir`  | `string`               |       No | Root directory for widget folders. Defaults to `src/widgets`. |

### Properties

#### `nativeServer`

```ts
readonly nativeServer: OfficialMcpServer
```

Provides direct access to the underlying `@modelcontextprotocol/sdk` `McpServer`.

This acts as an escape hatch when functionality from the official SDK is required.

#### config

```ts
readonly config: MCPServerConfig
```

The configuration supplied to the server.

#### http

```ts
get http(): HttpHandle | undefined
```

Returns the active HTTP server handle after an HTTP `listen()` call.

---

## tool()

Registers an MCP tool.

```ts
server.tool(definition, callback?)
```

Example:

```ts
server.tool(
  {
    name: "add",
    description: "Adds two numbers",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  },
  async ({ a, b }) => ({
    result: a + b,
  })
);
```

Returns the same `MCPServer` instance, allowing method chaining.

---

## prompt()

Registers an MCP prompt.

```ts
server.prompt(definition, callback?)
```

Example:

```ts
server.prompt(
  {
    name: "code-review",
    description: "Generate a code review prompt",
    arguments: [
      {
        name: "language",
        description: "Programming language",
        required: true,
      },
    ],
  },
  async ({ language }) => ({
    messages: [
      {
        role: "user",
        content: {
          type: "text",
          text: `Review this ${language} code.`,
        },
      },
    ],
  })
);
```

---

## resource()

Registers an MCP resource.

```ts
server.resource(definition, callback?)
```

Example:

```ts
server.resource(
  {
    uri: "config://app",
    name: "Application configuration",
  },
  async () => ({
    contents: [
      {
        uri: "config://app",
        text: JSON.stringify({ environment: "production" }),
      },
    ],
  })
);
```

---

## resourceTemplate()

Registers a parameterized resource template.

```ts
server.resourceTemplate(definition, callback?)
```

Example:

```ts
server.resourceTemplate(
  {
    uriTemplate: "users://{id}",
    name: "User",
  },
  async ({ id }) => ({
    contents: [
      {
        uri: `users://${id}`,
        text: JSON.stringify({ id }),
      },
    ],
  })
);
```

---

## widget()

Registers an HTML-based UI resource.

```ts
server.widget(definition, callback?)
```

> **Deprecated:** New projects should use the `widget` option on `server.tool()` instead. The method remains available for compatibility.

---

## refreshResource()

Notifies subscribed clients that a resource has new content.

```ts
await server.refreshResource(uri);
```

### Parameters

| Parameter | Type     | Description                     |
| --------- | -------- | ------------------------------- |
| `uri`     | `string` | URI of the resource to refresh. |

---

## refreshResources()

Requests clients to list resources again.

```ts
server.refreshResources();
```

---

## refreshTools()

Requests clients to list tools again.

```ts
server.refreshTools();
```

---

## refreshPrompts()

Requests clients to list prompts again.

```ts
server.refreshPrompts();
```

---

## mountRemote()

Mounts tools, prompts, and resources from other HTTP MCP servers onto the current server.

```ts
await server.mountRemote(remotes);
```

Remote names are exposed using the format:

```text
{alias}__{original}
```

Example:

```ts
await server.mountRemote({
  weather: {
    url: "https://example.com/mcp",
  },
});
```

---

## listen()

Starts the MCP server.

```ts
await server.listen(options?)
```

### ListenOptions

```ts
interface ListenOptions {
  transport?: "stdio" | "http";
  port?: number;
  host?: string;
  silent?: boolean;
}
```

| Property    | Type                | Default         | Description                              |
| ----------- | ------------------- | --------------- | ---------------------------------------- |
| `transport` | `"stdio" \| "http"` | `"stdio"`       | Server transport.                        |
| `port`      | `number`            | `3000` for HTTP | HTTP listening port.                     |
| `host`      | `string`            | `"localhost"`   | HTTP listening host.                     |
| `silent`    | `boolean`           | `false`         | Suppresses the startup URL log for HTTP. |

Example:

```ts
await server.listen({
  transport: "http",
  port: 4000,
});
```

### ListenResult

```ts
interface ListenResult {
  transport: "stdio" | "http";
  port?: number;
  host?: string;
  url?: string;
}
```

For HTTP, the result contains the actual bound port, host, and MCP endpoint URL.

---

## close()

Stops the server and closes mounted remote connections.

```ts
await server.close();
```

---

## parsePortFromArgv()

Reads a port from command-line arguments.

```ts
parsePortFromArgv(argv?: string[]): number | undefined
```

Both formats are supported:

```text
--port 4000
--port=4000
```

If multiple port arguments are supplied, the last valid occurrence wins.

---

# Response Helpers

The server entry point exports helpers for common tool responses.

```ts
import {
  text,
  markdown,
  image,
  object,
  error,
} from "mcpfy-sdk/server";
```

## text()

Creates a text content response.

```ts
text("Hello from MCPfy");
```

## markdown()

Creates a Markdown content response.

```ts
markdown("# Weather\n\nThe temperature is 24°C.");
```

## image()

Creates an image content response.

```ts
image("https://example.com/weather.png");
```

## object()

Creates a structured object response.

```ts
object({
  city: "Delhi",
  temperature: 24,
});
```

## error()

Creates an error response.

```ts
error("Unable to fetch weather data");
```

These helpers are useful for keeping tool handlers concise and consistent.

---

# Server Context

The server package exports:

```ts
ToolContext
SampleOptions
LogLevel
AskUrlOptions
```

`ToolContext` provides information and operations available to a tool during execution.

Example:

```ts
server.tool(
  {
    name: "example",
  },
  async (_input, ctx) => {
    ctx.log("Tool executed");

    return {
      success: true,
    };
  }
);
```

The server context also provides helpers for forwarding supported authentication headers and interacting with MCP capabilities.

---

# Widgets

The server entry point exports widget-related types and APIs:

```ts
UIResourceDefinition
WidgetCallback
WidgetContent
WidgetCsp
WidgetOptions
WidgetProtocol
DEFAULT_WIDGETS_DIR
```

The widget implementation supports multiple UI protocols, including:

* MCP-UI
* MCP Apps
* Apps SDK

Widgets can be attached to tools and prepared automatically when the server starts.

---

# Authentication

Authentication types and utilities are available from the server entry point:

```ts
import {
  jwksVerifier,
  oauthAuth0Provider,
  oauthWorkOSProvider,
} from "mcpfy-sdk/server";
```

## jwksVerifier()

Creates a JWT/JWKS-based bearer-token verifier.

```ts
jwksVerifier({
  issuer: "https://issuer.example.com",
  jwksUri: "https://issuer.example.com/.well-known/jwks.json",
  audience: "my-api",
});
```

### Options

```ts
interface JwksVerifierOptions {
  issuer: string;
  jwksUri: string;
  audience?: string;
}
```

The verifier validates the token signature and expected issuer/audience and extracts standard authentication information.

---

# mcpfy-sdk/client

The client entry point provides APIs for consuming MCP servers.

```ts
import {
  MCPClient,
  MCPSession,
  StdioConnector,
  HttpConnector,
  createConnectorFromConfig,
} from "mcpfy-sdk/client";
```

---

## MCPClient

```ts
class MCPClient
```

Creates and manages sessions for configured MCP servers.

### Configuration

```ts
interface MCPClientConfig {
  mcpServers: Record<string, ServerConfig>;
}
```

Example:

```ts
const client = new MCPClient({
  mcpServers: {
    weather: {
      transport: "stdio",
      command: "node",
      args: ["weather-server.js"],
    },
  },
});
```

---

## createSession()

Creates a session for a configured server.

```ts
const session = await client.createSession("weather");
```

If a session for the same server already exists, the existing session is returned.

---

## createAllSessions()

Creates sessions for all configured servers.

```ts
const sessions = await client.createAllSessions();
```

Returns:

```ts
Record<string, MCPSession>
```

---

## getSession()

Retrieves an existing session.

```ts
const session = client.getSession("weather");
```

Returns:

```ts
MCPSession | undefined
```

---

## closeAllSessions()

Closes every active session.

```ts
await client.closeAllSessions();
```

---

# Connectors

The client package provides:

```ts
BaseConnector
StdioConnector
HttpConnector
createConnectorFromConfig
```

Connectors are responsible for establishing communication with MCP servers.

Supported transport types include:

* stdio
* HTTP

Connector configuration is represented by:

```ts
ServerConfig
```

Use `createConnectorFromConfig()` when you need to construct a connector directly from a server configuration.

---

# MCPSession

`MCPSession` represents an active connection to an MCP server.

```ts
const session = await client.createSession("weather");
```

The session provides the higher-level operations used to interact with the connected MCP server.

---

# mcpfy-sdk/widget

The widget entry point provides React APIs for interactive MCP widgets.

```ts
import {
  HostRuntime,
  ThemeProvider,
  useHostContext,
  useHostProtocol,
  useToolPayload,
  useCallTool,
  useSendFollowUp,
  useOpenExternal,
  useLayoutMode,
  useHostTheme,
  useLinkedTool,
  useWidgetState,
  useViewState,
  useModelContext,
  useViewTool,
  HostImage,
} from "mcpfy-sdk/widget";
```

These APIs are documented in the **Widget React** guide.

---

# mcpfy-sdk/widget-bridge

The widget bridge entry point provides lower-level communication between widgets and their hosts.

```ts
import {
  connect,
  detectHostProtocol,
  postLink,
  postPrompt,
  postToolCall,
} from "mcpfy-sdk/widget-bridge";
```

It is primarily intended for widget integrations and advanced use cases where direct host communication is required.

---

# mcpfy-sdk/auth

The auth entry point provides OAuth client-side utilities.

```ts
import {
  NodeOAuthClientProvider,
  ensureAuthorized,
  OAuthSessionStore,
  FileKVStore,
} from "mcpfy-sdk/auth";
```

---

## NodeOAuthClientProvider

Provides a Node.js OAuth client implementation for MCP authentication flows.

```ts
new NodeOAuthClientProvider(options)
```

The provider implements the official MCP SDK's `OAuthClientProvider` interface.

---

## ensureAuthorized()

Ensures that the OAuth client has authorization before proceeding.

```ts
await ensureAuthorized(provider);
```

---

## OAuthSessionStore

Provides storage for OAuth session information.

```ts
const store = new OAuthSessionStore(...);
```

---

## FileKVStore

Provides a file-backed key-value store.

```ts
const store = new FileKVStore(...);
```

The storage implementation can be used by authentication components that need persistent local state.

---

# TypeScript Exports

mcpfy exposes TypeScript types for its public APIs so applications can maintain type safety throughout server, client, authentication, and widget integrations.

Common exported types include:

```ts
MCPServerConfig
ListenOptions
ListenResult

ToolDefinition
ToolCallback
ToolContext

PromptDefinition
PromptCallback

ResourceDefinition
ReadResourceCallback
FlatResourceTemplateDefinition
ReadResourceTemplateCallback

AuthConfig
AuthInfo

ServerConfig

UIResourceDefinition
WidgetCallback
WidgetContent
WidgetCsp
WidgetOptions
WidgetProtocol

HostCapabilities
LayoutMode
HostTheme
ToolPayload
CallToolHandle
HostEnv
ViewToolDefinition
ModelContextPublish
```

---

# Escape Hatch: Official MCP SDK

mcpfy wraps the official MCP SDK without preventing direct access to it.

```ts
const server = new MCPServer({
  name: "example",
  version: "1.0.0",
});

server.nativeServer;
```

Use `nativeServer` when an application needs functionality that is not directly exposed through mcpfy's higher-level API.

This design allows developers to start with the simplified mcpfy API while retaining access to the underlying MCP SDK when necessary.
