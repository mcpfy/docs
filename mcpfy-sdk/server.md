# MCP Server

`mcpfy-sdk/server` provides the server-side API for building MCP servers with tools, prompts, resources, resource templates, widgets, authentication, and multiple transports.

The main entry point is `MCPServer`.

---

## 1. Importing the Server API

```typescript
import { MCPServer } from "mcpfy-sdk/server";
```

Additional server APIs can be imported from the same entry point:

```typescript
import {
  MCPServer,
  jwksVerifier,
  text,
  markdown,
  image,
  object,
  error,
} from "mcpfy-sdk/server";
```

---

## 2. Creating an MCP Server

A server requires a name and version.

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});
```

The `MCPServer` class wraps the official MCP SDK server while providing a simpler declarative API.

The underlying native server remains accessible through:

```typescript
server.nativeServer
```

This provides an escape hatch for functionality that is not directly exposed by mcpfy.

---

## 3. Server Configuration

The server accepts an `MCPServerConfig` object.

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

### Configuration options

| Option        | Required | Description                            |
| ------------- | -------: | -------------------------------------- |
| `name`        |      Yes | Name advertised by the MCP server      |
| `version`     |      Yes | Server version                         |
| `description` |       No | Server description/instructions        |
| `basePath`    |       No | HTTP pathname for the MCP endpoint     |
| `icon`        |       No | Server icon                            |
| `auth`        |       No | HTTP authentication configuration      |
| `widgetsDir`  |       No | Root directory used for widget folders |

Example:

```typescript
const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
  description: "Provides weather-related MCP tools.",
  basePath: "/weather",
  widgetsDir: "src/widgets",
});
```

---

## 4. Server Capabilities

When `MCPServer` is created, mcpfy initializes the underlying official MCP server with support for:

* Logging
* Tools
* Prompts
* Resources
* Resource subscriptions
* List-change notifications

The server advertises list-change support for tools, prompts, and resources.

Resource subscriptions are also enabled automatically.

---

## 5. Registering Tools

Tools are registered using:

```typescript
server.tool()
```

Example:

```typescript
server.tool({
  name: "hello",
  description: "Say hello",
}, async () => {
  return {
    content: [
      {
        type: "text",
        text: "Hello!",
      },
    ],
  };
});
```

The tool API is documented in detail in `tools.md`.

---

## 6. Registering Prompts

Prompts are registered using:

```typescript
server.prompt()
```

Example:

```typescript
server.prompt({
  name: "code-review",
  description: "Review a piece of code",
}, async (params) => {
  // Return prompt messages
});
```

The prompt API is documented in detail in `prompts.md`.

---

## 7. Registering Resources

Resources are registered using:

```typescript
server.resource()
```

Example:

```typescript
server.resource({
  name: "config",
  uri: "config://app",
}, async () => {
  // Return resource contents
});
```

Resource templates are supported through:

```typescript
server.resourceTemplate()
```

The complete resource API is documented in `resources.md`.

---

## 8. Registering Widgets

mcpfy supports interactive MCP widgets.

Widgets can be registered through:

```typescript
server.widget()
```

However, the `widget()` method is deprecated for new widget implementations.

The recommended approach is to specify a widget folder through a tool definition:

```typescript
server.tool({
  name: "weather",
  widget: "weather",
});
```

The widget system supports multiple protocols, including MCP-UI, MCP Apps, and Apps SDK integrations.

Widget functionality is documented in `widgets.md` and `widget-react.md`.

---

# 9. Starting the Server

The server is started using:

```typescript
await server.listen();
```

By default, mcpfy uses **stdio** transport.

```typescript
await server.listen();
```

is equivalent to:

```typescript
await server.listen({
  transport: "stdio",
});
```

Stdio is the typical transport used when an MCP host launches the server as a local process.

---

# 10. HTTP Transport

An MCP server can also be started using HTTP:

```typescript
await server.listen({
  transport: "http",
});
```

The default HTTP port is:

```text
3000
```

The default host is:

```text
localhost
```

Therefore:

```typescript
await server.listen({
  transport: "http",
});
```

normally creates an MCP endpoint at:

```text
http://localhost:3000/mcp
```

---

## 11. HTTP Configuration

`ListenOptions` controls how the server starts.

```typescript
interface ListenOptions {
  transport?: "stdio" | "http";
  port?: number;
  host?: string;
  silent?: boolean;
}
```

### `transport`

Selects the server transport.

```typescript
transport: "stdio"
```

or:

```typescript
transport: "http"
```

The default is:

```typescript
"stdio"
```

---

### `port`

Specifies the HTTP port.

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

The server will listen on:

```text
http://localhost:4000/mcp
```

Passing `0` allows the operating system to select an available port:

```typescript
await server.listen({
  transport: "http",
  port: 0,
});
```

The actual assigned port is returned by `listen()`.

---

### `host`

Specifies the HTTP listening host.

```typescript
await server.listen({
  transport: "http",
  host: "0.0.0.0",
});
```

The default is:

```text
localhost
```

---

### `silent`

Controls whether the HTTP startup URL is printed.

```typescript
await server.listen({
  transport: "http",
  silent: true,
});
```

The default is:

```typescript
false
```

---

# 12. HTTP Port Resolution

When using HTTP transport, mcpfy determines the port using the following priority:

```text
1. listen({ port })
2. --port N / --port=N
3. process.env.PORT
4. 3000
```

For example:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

takes priority over the environment variable.

The command-line argument:

```bash
node server.js --port 5000
```

is also supported.

The `--port=N` form is supported as well:

```bash
node server.js --port=5000
```

---

## 13. Parsing a Port from Arguments

The helper:

```typescript
parsePortFromArgv()
```

is exported from `mcpfy-sdk/server`.

Example:

```typescript
import { parsePortFromArgv } from "mcpfy-sdk/server";

const port = parsePortFromArgv();
```

It supports:

```text
--port 4000
```

and:

```text
--port=4000
```

If multiple port arguments are present, the last valid occurrence wins.

The function returns:

```typescript
number | undefined
```

---

# 14. Environment Port

When no explicit port or command-line port is provided, mcpfy checks:

```text
process.env.PORT
```

For example:

```powershell
$env:PORT = "8080"
```

Then:

```typescript
await server.listen({
  transport: "http",
});
```

uses port `8080`.

If the environment variable is missing or invalid, mcpfy falls back to port `3000`.

---

# 15. Server Listen Result

`listen()` returns a `ListenResult`.

```typescript
interface ListenResult {
  transport: "stdio" | "http";
  port?: number;
  host?: string;
  url?: string;
}
```

For stdio:

```typescript
const result = await server.listen({
  transport: "stdio",
});
```

The result is:

```typescript
{
  transport: "stdio"
}
```

For HTTP:

```typescript
const result = await server.listen({
  transport: "http",
  port: 4000,
});
```

The result contains:

```typescript
{
  transport: "http",
  port: 4000,
  host: "localhost",
  url: "http://localhost:4000/mcp"
}
```

---

# 16. Custom MCP HTTP Path

The HTTP MCP endpoint defaults to:

```text
/mcp
```

A custom path can be configured using `basePath`.

```typescript
const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
  basePath: "/weather",
});
```

Starting the HTTP server:

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});
```

produces an endpoint at:

```text
http://localhost:3000/weather
```

The configured path is normalized internally before being passed to the HTTP transport.

---

# 17. Server Icons

An MCP server can advertise an icon through the `icon` configuration.

```typescript
const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
  icon: "./icon.png",
});
```

The icon can be provided as:

* A remote URL
* A `data:` URI
* A local file path
* A `file:` URL
* A `ServerIcon` object

Local files are converted to `data:` URIs so MCP clients can display them.

---

# 18. Authentication

HTTP servers can require authentication using the `auth` configuration.

```typescript
const server = new MCPServer({
  name: "secure-server",
  version: "1.0.0",
  auth: authConfig,
});
```

Authentication is applied to HTTP transport.

For example:

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});
```

Clients must provide the appropriate bearer token.

JWT/JWKS authentication can be configured using:

```typescript
import { jwksVerifier } from "mcpfy-sdk/server";
```

Detailed authentication documentation is provided in `authentication.md`.

---

# 19. Accessing the Native MCP Server

The underlying official MCP server is exposed through:

```typescript
server.nativeServer
```

Example:

```typescript
const nativeServer = server.nativeServer;
```

This allows advanced users to access functionality directly from the official MCP SDK.

mcpfy therefore acts as a higher-level API rather than preventing access to the underlying MCP implementation.

---

# 20. Refreshing Resources

mcpfy provides resource refresh functionality.

To notify subscribed clients that a specific resource has changed:

```typescript
await server.refreshResource("config://app");
```

Example:

```typescript
await server.refreshResource("data://users");
```

This tells subscribed clients that new content is available for the specified resource URI.

---

# 21. Refreshing the Resource List

To tell clients to request the resource list again:

```typescript
server.refreshResources();
```

This is useful when resources are dynamically added or removed.

---

# 22. Refreshing the Tool List

To tell clients to request the tool list again:

```typescript
server.refreshTools();
```

This is useful when the available tools change during the server's lifetime.

---

# 23. Refreshing the Prompt List

To tell clients to request the prompt list again:

```typescript
server.refreshPrompts();
```

This notifies clients that they should refresh their available prompt definitions.

---

# 24. Mounting Remote MCP Servers

mcpfy can expose tools, prompts, and resources from other HTTP MCP servers through the current server.

This is done using:

```typescript
server.mountRemote()
```

Example:

```typescript
await server.mountRemote({
  weather: {
    // remote server configuration
  },
});
```

The remote server's tools and prompts are exposed using an alias-based naming scheme.

For example:

```text
weather__get_forecast
```

represents a remote tool named:

```text
get_forecast
```

mounted using the alias:

```text
weather
```

This allows multiple remote MCP servers to be combined behind one MCP server.

---

# 25. Closing the Server

A running server can be closed using:

```typescript
await server.close();
```

`close()` performs cleanup for:

1. The HTTP server, if running.
2. Mounted remote server connections.
3. The underlying native MCP server.

Example:

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});

// ... server is running

await server.close();
```

After closing, the HTTP handle and mounted remote connections are cleared.

---

# 26. HTTP Server Handle

After starting an HTTP server, additional information is available through:

```typescript
server.http
```

Example:

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});

console.log(server.http);
```

The value is an `HttpHandle` while the HTTP server is running.

It becomes undefined after the server is closed.

---

# 27. Complete HTTP Server Example

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "example-server",
  version: "1.0.0",
  description: "Example MCP server",
  basePath: "/mcp",
});

server.tool(
  {
    name: "hello",
    description: "Returns a greeting",
  },
  async () => {
    return text("Hello from mcpfy!");
  }
);

const result = await server.listen({
  transport: "http",
  port: 3000,
});

console.log(`MCP server running at ${result.url}`);
```

The server is available at:

```text
http://localhost:3000/mcp
```

---

# 28. Complete Stdio Server Example

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "stdio-server",
  version: "1.0.0",
});

server.tool(
  {
    name: "hello",
    description: "Returns a greeting",
  },
  async () => {
    return text("Hello from the MCP server!");
  }
);

await server.listen();
```

Because stdio is the default transport, the final call can simply be:

```typescript
await server.listen();
```

---

# 29. Recommended Server Structure

A larger MCP application can organize the server separately from its tools and other features.

Example:

```text
my-mcp-server/
├── src/
│   ├── index.ts
│   ├── tools/
│   │   ├── weather.ts
│   │   └── search.ts
│   ├── prompts/
│   │   └── assistant.ts
│   ├── resources/
│   │   └── documentation.ts
│   └── widgets/
│       └── weather/
├── package.json
└── tsconfig.json
```

The main server can then initialize and register the individual components.

---

# 30. Public Server API

The `mcpfy-sdk/server` entry point exports the following major APIs.

### Server

```typescript
MCPServer
parsePortFromArgv
```

### Server types

```typescript
MCPServerConfig
ListenOptions
ListenResult
ServerIcon
HttpHandle
```

### Tools

```typescript
ToolDefinition
ToolCallback
```

### Prompts

```typescript
PromptDefinition
PromptCallback
```

### Resources

```typescript
ResourceDefinition
ReadResourceCallback
FlatResourceTemplateDefinition
ReadResourceTemplateCallback
```

### Context

```typescript
ToolContext
SampleOptions
LogLevel
AskUrlOptions
```

### Responses

```typescript
text
markdown
image
object
error
```

### Widgets

```typescript
UIResourceDefinition
WidgetCallback
WidgetContent
WidgetCsp
WidgetOptions
WidgetProtocol
DEFAULT_WIDGETS_DIR
```

### Authentication

```typescript
AuthConfig
AuthInfo
jwksVerifier
JwksVerifierOptions
oauthAuth0Provider
oauthWorkOSProvider
```

### Remote servers

```typescript
RemoteServerConfig
```

---

# 31. Server API Summary

| API                         | Purpose                                    |
| ---------------------------- | ------------------------------------------- |
| `new MCPServer()`           | Create an MCP server                       |
| `server.tool()`             | Register an MCP tool                       |
| `server.prompt()`           | Register an MCP prompt                     |
| `server.resource()`         | Register an MCP resource                   |
| `server.resourceTemplate()` | Register a resource template               |
| `server.widget()`           | Register a legacy HTML widget              |
| `server.listen()`           | Start the server                           |
| `server.close()`            | Stop and clean up the server               |
| `server.refreshResource()`  | Notify subscribers that a resource changed |
| `server.refreshResources()` | Refresh the resource list                  |
| `server.refreshTools()`     | Refresh the tool list                      |
| `server.refreshPrompts()`   | Refresh the prompt list                    |
| `server.mountRemote()`      | Mount another HTTP MCP server              |
| `server.nativeServer`       | Access the underlying MCP SDK server       |
| `server.http`               | Access the active HTTP handle              |

---

# 32. Recommended Usage Pattern

A typical mcpfy server follows this pattern:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "my-mcp-server",
  version: "1.0.0",
});

// Register tools, prompts, resources, and widgets.

await server.listen({
  transport: "http",
  port: 3000,
});
```

For local MCP hosts such as Claude Desktop or Claude Code, stdio is generally used:

```typescript
await server.listen({
  transport: "stdio",
});
```

For remotely accessible MCP servers, HTTP transport can be used:

```typescript
await server.listen({
  transport: "http",
  port: 3000,
});
```

The same `MCPServer` abstraction supports both approaches, allowing the application logic to remain independent of the selected transport.