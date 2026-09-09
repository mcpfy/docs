
# mcpfy Architecture

## 1. Overview

mcpfy is a lightweight TypeScript SDK built on top of the official Model Context Protocol SDK. Its architecture is designed to simplify the development of MCP servers and clients while preserving access to the underlying MCP implementation.

The SDK provides abstractions for:

- MCP servers
- MCP clients
- Tools
- Prompts
- Resources
- Resource templates
- Authentication
- HTTP and stdio transports
- Remote MCP server mounting
- Interactive widgets
- MCP-UI
- MCP Apps
- Apps SDK
- React-based widget runtimes
- Host communication and capabilities
- Resource subscriptions and refresh
- OAuth/JWT authentication

The overall architecture can be viewed as:

```
                    ┌──────────────────────────┐
                    │        Application       │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │        mcpfy SDK         │
                    ├──────────────────────────┤
                    │ MCPServer / MCPClient    │
                    │ Tools / Prompts          │
                    │ Resources                │
                    │ Authentication            │
                    │ Widgets                   │
                    └────────────┬─────────────┘
                                 │
                  ┌──────────────┴──────────────┐
                  ▼                             ▼
       ┌────────────────────┐         ┌────────────────────┐
       │ Official MCP SDK   │         │ Widget Protocols   │
       │ @modelcontext...   │         │ MCP-UI              │
       │                    │         │ MCP Apps            │
       │                    │         │ Apps SDK            │
       └────────────────────┘         └────────────────────┘
```

The core design principle is abstraction without isolation. mcpfy provides a simpler API for common operations but exposes the underlying official MCP server through `nativeServer` whenever advanced functionality is required.

## 2. Package Structure

The main package is located at:

```
typescript/packages/mcpfy/
```

Its source code is organized into functional modules:

```
src/
├── index.ts
│
├── auth/
│   ├── file-kv-store.ts
│   ├── index.ts
│   ├── kv-store.ts
│   ├── node-oauth-provider.ts
│   └── oauth-session-store.ts
│
├── cli/
│   └── mcpfy.ts
│
├── client/
│   ├── connectors.ts
│   ├── index.ts
│   ├── mcp-client.ts
│   └── session.ts
│
├── client-widget/
│   ├── apps-sdk-types.ts
│   ├── index.ts
│   ├── mcp-apps.ts
│   └── mcp-ui-actions.ts
│
├── server/
│   ├── auth/
│   ├── widgets/
│   ├── context.ts
│   ├── icon.ts
│   ├── index.ts
│   ├── json-schema.ts
│   ├── mcp-server.ts
│   ├── mount-remote.ts
│   ├── prompts.ts
│   ├── refresh.ts
│   ├── resources.ts
│   ├── subscriptions.ts
│   ├── tools.ts
│   └── transport.ts
│
├── shared/
│   └── response-helpers.ts
│
└── widget-react/
    ├── host-capabilities.ts
    ├── index.ts
    └── runtime.tsx
```

Each module has a focused responsibility rather than placing all functionality inside the main server or client implementation.

## 3. Server Architecture

The central server abstraction is the `MCPServer` class.

It wraps the official:

```
McpServer
```

from:

```
@modelcontextprotocol/sdk/server/mcp.js
```

The wrapper provides a simpler API while maintaining access to the native implementation.

The important relationship is:

```
Application
     │
     ▼
  MCPServer
     │
     ├── Tools
     ├── Prompts
     ├── Resources
     ├── Widgets
     ├── Authentication
     ├── Refresh / subscriptions
     ├── Remote mounting
     └── Transport
             │
             ▼
     Official McpServer
```

The underlying server is available through:

```
server.nativeServer
```

This provides an escape hatch for functionality that is not directly abstracted by mcpfy.

## 4. MCPServer Initialization

`MCPServer` accepts an `MCPServerConfig` object.

The configuration includes:

```typescript
export interface MCPServerConfig {
  name: string;
  version: string;
  description?: string;
  basePath?: string;
  icon?: string | ServerIcon;
  auth?: AuthConfig;
  widgetsDir?: string;
}
```

The constructor creates the underlying official MCP server:

```typescript
this.nativeServer = new OfficialMcpServer(...)
```

The server is initialized with:

- name
- version
- title
- icons
- description/instructions
- logging capability
- tool list-change notifications
- prompt list-change notifications
- resource subscriptions
- resource list-change notifications

The widget registry and resource subscriptions are also configured during construction.

## 5. Tool Registration Architecture

Tools are registered through:

```typescript
server.tool(...)
```

The public method delegates the actual registration to:

```typescript
registerTool(...)
```

located in:

```
src/server/tools.ts
```

The architecture is:

```
server.tool()
      │
      ▼
registerTool()
      │
      ▼
Official McpServer.registerTool()
```

This separation keeps `MCPServer` focused on the public API while tool-specific logic remains inside the tools module.

A tool definition may contain information such as:

- name
- title
- description
- input schema
- output schema
- metadata

The callback receives the validated input and tool context.

## 6. Tool Context

Tool execution can access contextual information through `ToolContext`.

The context is constructed using:

```typescript
buildToolContext(...)
```

from:

```
src/server/context.ts
```

This allows tool implementations to interact with MCP functionality without directly depending on the internal server implementation.

The context can provide functionality related to:

- logging
- authentication information
- requesting model sampling
- forwarding authentication headers
- sending requests to external services
- URL-related interactions

This creates a clean separation between the business logic of a tool and the underlying MCP request/response infrastructure.

## 7. Prompt Architecture

Prompts are registered using:

```typescript
server.prompt(...)
```

Internally this delegates to:

```typescript
registerPrompt(...)
```

in:

```
src/server/prompts.ts
```

The flow is:

```
Application
    │
    ▼
server.prompt()
    │
    ▼
registerPrompt()
    │
    ▼
Official MCP prompt registration
```

Prompt registration is intentionally implemented separately from tools because MCP treats prompts as a distinct primitive.

The SDK also supports prompt refresh notifications through:

```typescript
server.refreshPrompts()
```

## 8. Resource Architecture

Resources are registered through:

```typescript
server.resource(...)
```

Resource templates are registered through:

```typescript
server.resourceTemplate(...)
```

The resource implementation is contained in:

```
src/server/resources.ts
```

The architecture supports:

```
Resources
   │
   ├── Static resources
   │
   └── Resource templates
```

Resource subscriptions are enabled when the server is initialized.

This allows clients to subscribe to resource updates.

The server exposes:

```typescript
server.refreshResource(uri)
```

for notifying subscribed clients that a specific resource has changed.

It also provides:

```typescript
server.refreshResources()
```

to request that clients refresh their resource listing.

## 9. Refresh and Change Notifications

The SDK contains a dedicated:

```
src/server/refresh.ts
```

module.

It handles notifications for changes to:

- resources
- tools
- prompts

The public API includes:

```typescript
server.refreshResource(uri)
server.refreshResources()
server.refreshTools()
server.refreshPrompts()
```

The design separates the registration of MCP primitives from the notification mechanism.

For example:

```
Tool registration
      │
      ▼
Official MCP server
      │
      ▼
Tool list changes
      │
      ▼
refreshTools()
      │
      ▼
Notify connected clients
```

This makes dynamic MCP servers easier to implement.

## 10. Resource Subscriptions

Resource subscriptions are enabled through:

```
src/server/subscriptions.ts
```

During server initialization:

```typescript
enableResourceSubscriptions(this.nativeServer)
```

is called.

This provides the infrastructure required for clients to subscribe to resource updates.

The architecture is:

```
Client
  │
  │ subscribe
  ▼
MCPServer
  │
  ▼
Resource subscription system
  │
  │ resource changes
  ▼
Client notification
```

This is particularly useful for MCP servers where resource content can change over time.

## 11. Transport Architecture

The SDK supports two server transports:

- `"stdio"`
- `"http"`

Transport logic is implemented in:

```
src/server/transport.ts
```

The server's public API is:

```typescript
await server.listen({
  transport: "stdio"
});
```

or:

```typescript
await server.listen({
  transport: "http",
  port: 3000
});
```

The architecture is:

```
                    MCPServer.listen()
                           │
                 ┌─────────┴─────────┐
                 ▼                   ▼
              stdio                 HTTP
                 │                   │
                 ▼                   ▼
           startStdio()          startHttp()
                 │                   │
                 ▼                   ▼
          MCP transport        HTTP transport
```

The default transport is stdio.

This is useful for MCP hosts that launch servers as local processes.

HTTP transport is intended for network-accessible MCP servers.

## 12. HTTP Path and Port Resolution

The HTTP MCP endpoint defaults to:

```
/mcp
```

A custom path can be configured using:

```
basePath
```

For example:

```typescript
new MCPServer({
  name: "weather",
  version: "1.0.0",
  basePath: "/weather"
});
```

The resulting endpoint becomes conceptually:

```
http://localhost:<port>/weather
```

Port resolution follows this priority:

```
Explicit listen() port
        ↓
--port CLI argument
        ↓
PORT environment variable
        ↓
3000
```

The helper:

```typescript
parsePortFromArgv()
```

supports both:

```
--port 4000
```

and:

```
--port=4000
```

The last valid occurrence takes precedence.

## 13. Authentication Architecture

Authentication is implemented as a separate server subsystem.

Relevant files include:

```
src/server/auth/
```

The architecture separates:

- Token extraction
- Token verification
- Authentication information
- HTTP transport enforcement

The basic flow is:

```
HTTP Request
     │
     ▼
Authorization header
     │
     ▼
Bearer token extraction
     │
     ▼
Configured verifier
     │
     ├── valid ──► AuthInfo
     │
     └── invalid ─► authentication failure
```

Authentication is only applied to HTTP transport.

## 14. Bearer Token Middleware

The file:

```
src/server/auth/middleware.ts
```

contains the framework-free authentication check.

It reads:

```
Authorization: Bearer <token>
```

The function:

```typescript
checkAuth(req, config)
```

returns either:

```typescript
{ ok: true, auth: AuthInfo }
```

or:

```typescript
{ ok: false }
```

This keeps authentication independent from any particular HTTP framework.

## 15. JWT and JWKS Verification

The SDK provides a generic JWT verification implementation through:

```typescript
jwksVerifier(...)
```

located in:

```
src/server/auth/jwks-verifier.ts
```

It uses the jose library.

The verifier accepts:

```typescript
export interface JwksVerifierOptions {
  issuer: string;
  jwksUri: string;
  audience?: string;
}
```

The verification process is:

```
Bearer JWT
    │
    ▼
Remote JWKS
    │
    ▼
JWT signature verification
    │
    ▼
Issuer validation
    │
    ▼
Audience validation
    │
    ▼
AuthInfo
```

The JWKS endpoint is managed using `createRemoteJWKSet`, which provides remote key retrieval and caching behavior.

The verifier also extracts the OAuth-style scope claim when available.

For example:

```
scope = "read write profile"
```

is converted into:

```json
["read", "write", "profile"]
```

## 16. Authentication Provider Abstraction

The server authentication system uses the `AuthConfig` abstraction rather than coupling the server to a specific identity provider.

This allows authentication to be implemented using:

- custom header verification
- JWT/JWKS verification
- OAuth providers
- provider-specific presets

The architecture therefore remains provider-independent.

## 17. Client Architecture

The client implementation is located in:

```
src/client/
```

The main abstraction is:

```
MCPClient
```

The client manages multiple named MCP server connections.

Its configuration is:

```typescript
export interface MCPClientConfig {
  mcpServers: Record<string, ServerConfig>;
}
```

The architecture is:

```
MCPClient
    │
    ├── Server A
    │      │
    │      ▼
    │   Connector
    │      │
    │      ▼
    │   MCPSession
    │
    ├── Server B
    │      │
    │      ▼
    │   Connector
    │      │
    │      ▼
    │   MCPSession
    │
    └── Server C
```

## 18. Client Session Management

`MCPClient` maintains sessions in:

```typescript
private readonly sessions = new Map<string, MCPSession>();
```

When:

```typescript
createSession(name)
```

is called, the client:

1. Checks whether a session already exists.
2. Finds the server configuration.
3. Creates the appropriate connector.
4. Connects to the server.
5. Creates an `MCPSession`.
6. Stores the session.
7. Returns the session.

This prevents unnecessary duplicate connections.

The client also provides:

```typescript
createAllSessions()
```

which creates sessions for all configured servers.

## 19. Connector Architecture

Connectors are implemented in:

```
src/client/connectors.ts
```

The exported connector abstractions include:

- `BaseConnector`
- `StdioConnector`
- `HttpConnector`
- `createConnectorFromConfig`

The architecture allows the client to remain independent of the transport.

```
MCPClient
    │
    ▼
createConnectorFromConfig()
    │
    ├── stdio configuration
    │       ▼
    │   StdioConnector
    │
    └── HTTP configuration
            ▼
        HttpConnector
```

This makes adding or changing transport-specific behavior easier without modifying `MCPClient`.

## 20. MCP Session Architecture

The session abstraction is implemented in:

```
src/client/session.ts
```

A session represents an active logical connection to an MCP server.

The relationship is:

```
MCPClient
   │
   ▼
Connector
   │
   ▼
MCPSession
   │
   ▼
MCP operations
```

The session hides lower-level connection management from the application.

## 21. Remote MCP Server Mounting

mcpfy can expose functionality from other HTTP MCP servers through:

```typescript
server.mountRemote(...)
```

The implementation is located in:

```
src/server/mount-remote.ts
```

The architecture is:

```
Remote MCP Server A ──┐
                      │
Remote MCP Server B ──┼──► mcpfy Server ──► Client
                      │
Remote MCP Server C ──┘
```

Remote tools and prompts are re-exposed through the local server.

Names are prefixed using the configured alias:

```
{alias}__{original}
```

This provides a simple way to compose multiple MCP servers into a single MCP endpoint.

## 22. Widget Architecture

Widgets are one of the major architectural components of mcpfy.

The widget subsystem is located under:

```
src/server/widgets/
```

The SDK supports multiple widget protocols through a common abstraction.

The supported protocol integrations include:

- MCP-UI
- MCP Apps
- Apps SDK

The architecture is:

```
                  Widget Definition
                         │
                         ▼
                  Widget Registry
                         │
                         ▼
                 Widget Preparation
                         │
                         ▼
                Protocol Adapters
                 ┌───────┼───────┐
                 ▼       ▼       ▼
              MCP-UI  MCP Apps  Apps SDK
```

This allows application developers to define a widget once while the SDK handles protocol-specific integration.

## 23. Widget Registration

Widgets can be registered through the server's widget API.

Internally:

```typescript
registerWidget(...)
```

performs the registration.

A widget definition can contain:

- name
- title
- description
- input schema
- content
- protocols
- CSP configuration
- size configuration
- callback

The callback is executed when the corresponding MCP tool is called.

## 24. Widgets as MCP Tools

An important architectural decision is that widgets are connected to MCP tools.

When a widget is registered, mcpfy registers an MCP tool with the widget's name.

The execution flow is:

```
Host / Model
      │
      ▼
MCP Tool Call
      │
      ▼
Widget callback
      │
      ├── structured data
      │
      └── widget content
      │
      ▼
MCP Tool Result
      │
      ▼
Widget Host
```

The callback result is returned as:

```
structuredContent
```

and a textual representation is also generated.

When MCP-UI is enabled, an additional MCP-UI content block is attached.

This architecture allows the same tool call to provide both machine-readable data and UI-oriented content.

## 25. Widget Registry

Widget registry functionality is implemented in:

```
src/server/widgets/registry.ts
```

The registry is configured when the `MCPServer` is created.

The default widget directory is:

```
src/widgets
```

Applications can override this using:

```
widgetsDir
```

The registry allows widget folders to be discovered and prepared before the server begins accepting requests.

## 26. Widget Preparation and Bundling

Widget preparation is handled by:

```
src/server/widgets/prepare.ts
```

Supporting functionality is distributed across:

```
bundle.ts
vite-plugin.ts
html.ts
html-shell.ts
```

This subsystem prepares React/widget applications for serving through MCP.

The conceptual pipeline is:

```
Widget Source
     │
     ▼
Vite / React build
     │
     ▼
Widget bundle
     │
     ▼
HTML shell
     │
     ▼
MCP widget resource
```

This keeps frontend build concerns separate from the core MCP server implementation.

## 27. Widget Security and CSP

Widget content can specify Content Security Policy settings.

The implementation is located in:

```
src/server/widgets/csp.ts
```

The purpose of CSP configuration is to control which external resources a widget is allowed to access.

This is particularly important because widgets execute UI code in a host-controlled environment.

The architecture therefore separates:

```
Widget UI
   │
   ├── content
   ├── size
   └── CSP
```

rather than embedding security configuration directly into application components.

## 28. Client Widget Architecture

The client-side widget communication layer is located in:

```
src/client-widget/
```

It handles communication between the widget and the host.

The SDK detects the environment and chooses the appropriate protocol.

Conceptually:

```
Widget
  │
  ▼
mcpfy Widget Bridge
  │
  ├── Apps SDK
  ├── MCP Apps
  └── MCP-UI / iframe
```

This abstraction prevents individual React components from having to implement protocol-specific communication themselves.

## 29. React Widget Runtime

The React-specific runtime is implemented in:

```
src/widget-react/runtime.tsx
```

The main component is:

```jsx
<HostRuntime />
```

It creates the runtime environment used by widget hooks.

The runtime manages:

- host connection
- protocol detection
- tool input
- tool output
- pending state
- errors
- theme
- locale
- platform
- layout mode
- widget state
- model context
- view tools
- external links
- follow-up messages

## 30. Host Runtime Architecture

The runtime creates an internal adapter around the detected host.

Conceptually:

```
React Widget
     │
     ▼
HostRuntime
     │
     ▼
HostAdapter
     │
     ├── callTool()
     ├── sendFollowUp()
     ├── openExternal()
     ├── requestLayoutMode()
     ├── setWidgetState()
     ├── publishModelContext()
     └── registerViewTool()
```

The adapter provides one common interface while internally selecting the correct host implementation.

## 31. Protocol Detection

The widget runtime uses:

```typescript
detectHostProtocol()
```

to determine the environment.

The possible protocols include:

- apps-sdk
- mcp-apps
- mcp-ui
- none

The runtime then chooses the appropriate communication mechanism.

This allows the same React widget to operate across supported MCP host environments without rewriting its UI logic.

## 32. Tool Communication from Widgets

The hook:

```typescript
useCallTool()
```

allows a widget to call an MCP tool.

The runtime first attempts to use the host's native tool API.

For example, with Apps SDK:

```typescript
openai.callTool()
```

With MCP Apps:

```typescript
app.callServerTool()
```

If those mechanisms are unavailable, the runtime falls back to the corresponding messaging mechanism.

Conceptually:

```
useCallTool()
      │
      ▼
HostAdapter.callTool()
      │
      ├── Apps SDK
      ├── MCP Apps
      └── MCP-UI fallback
```

This is a key part of the protocol abstraction.

## 33. Widget State

The runtime supports persistent widget state through:

```typescript
useWidgetState()
```

and:

```typescript
useViewState()
```

Widget state is represented as:

```typescript
Record<string, unknown>
```

The runtime can restore state supplied by the host and persist new state through the appropriate protocol.

The architecture is:

```
React State
    │
    ▼
HostAdapter
    │
    ├── Apps SDK widget state
    │
    └── MCP Apps / model context
```

This allows widgets to maintain useful state across host interactions.

## 34. Model Context

The hook:

```typescript
useModelContext()
```

provides a mechanism for publishing information back to the model when supported by the host.

The runtime uses:

```typescript
publishModelContext()
```

internally.

For MCP Apps, this can use:

```
updateModelContext
```

For supported Apps SDK behavior, structured state can also be synchronized through the host.

This enables a widget to expose relevant UI state or user actions back to the model.

## 35. View Tools

The runtime supports tools registered directly by a mounted view through:

```typescript
useViewTool()
```

This is available when the host supports view tools.

The flow is:

```
React Component
      │
      ▼
useViewTool()
      │
      ▼
HostAdapter.registerViewTool()
      │
      ▼
Host
      │
      ▼
Tool invocation
      │
      ▼
React handler
```

On hosts that do not support view tools, the registration becomes a no-op.

This capability-based design prevents unsupported host features from causing runtime failures.

## 36. Host Capabilities

The widget runtime determines supported capabilities using:

```
host-capabilities.ts
```

Capabilities can include functionality such as:

- tool calling
- display mode support
- model context
- view tools
- host-specific features

Instead of assuming that every host supports every feature, the runtime checks capabilities before performing protocol-specific operations.

This makes the widget layer more portable.

## 37. Theme Management

The React runtime supports:

```typescript
HostTheme = "light" | "dark"
```

The initial theme is determined from the browser's preferred color scheme when no host-provided theme is available.

`ThemeProvider` stores the active theme and updates:

```
document.documentElement.dataset.mcpfyTheme
```

when the theme changes.

The hook:

```typescript
useHostTheme()
```

allows widget components to access the active theme.

## 38. Layout Management

Widgets can interact with host display modes through:

```typescript
useLayoutMode()
```

The runtime tracks:

- current layout mode
- available layout modes

and provides:

```typescript
request(mode)
```

to request a different layout.

The actual implementation is delegated to the host:

```
Apps SDK
    │
    └── requestDisplayMode()

MCP Apps
    │
    └── requestDisplayMode()

Fallback
    │
    └── local state update
```

## 39. External Links and Follow-Up Messages

The widget runtime provides:

```typescript
useOpenExternal()
```

and:

```typescript
useSendFollowUp()
```

These operations are also abstracted through `HostAdapter`.

For example:

```
useOpenExternal()
      │
      ▼
HostAdapter.openExternal()
      │
      ├── Apps SDK
      ├── MCP Apps
      └── MCP-UI fallback
```

The same pattern is used for sending follow-up messages.

## 40. Widget Size Reporting

The React runtime automatically reports widget dimensions to its parent.

It calculates the document's:

- scrollHeight
- scrollWidth

and sends:

```
ui-size-change
```

messages to the parent window.

A `ResizeObserver` is used to detect changes.

The architecture is:

```
Widget DOM
    │
    ▼
ResizeObserver
    │
    ▼
calculate dimensions
    │
    ▼
postMessage()
    │
    ▼
Widget host
```

This allows hosts to dynamically adapt the space allocated to a widget.

## 41. Shared Response Helpers

Common MCP response helpers are implemented in:

```
src/shared/response-helpers.ts
```

The server package exports helpers including:

- text
- markdown
- image
- object
- error

These provide a consistent way to construct common MCP tool responses.

The goal is to reduce repetitive response construction code while keeping the resulting data compatible with the underlying MCP SDK.

## 42. Authentication Client Support

Client-side authentication functionality is located under:

```
src/auth/
```

The module contains:

```
node-oauth-provider.ts
oauth-session-store.ts
file-kv-store.ts
kv-store.ts
```

This subsystem provides reusable OAuth-related client functionality and persistence abstractions.

The key components include:

- `NodeOAuthClientProvider`
- `OAuthSessionStore`
- `FileKVStore`
- `KVStore`

The architecture separates OAuth behavior from persistence.

## 43. Persistence Abstraction

The `KVStore` abstraction provides a simple key-value persistence interface.

The file-based implementation:

```
file-kv-store.ts
```

provides a Node.js-compatible storage mechanism.

This design allows the OAuth session layer to depend on an abstract storage interface rather than directly depending on the filesystem.

Conceptually:

```
OAuthSessionStore
       │
       ▼
     KVStore
       │
       ├── FileKVStore
       └── Future implementations
```

## 44. CLI Architecture

The CLI entry point is:

```
src/cli/mcpfy.ts
```

The package exposes the executable:

```
mcpfy
```

through the bin field of `package.json`.

The CLI can be used to work with MCP server execution and command-line configuration such as HTTP ports.

This keeps command-line concerns separate from the core SDK classes.

## 45. Public API Architecture

The package uses multiple public entry points.

The main entry point is:

```
mcpfy-sdk
```

Additional entry points include:

```
mcpfy-sdk/server
mcpfy-sdk/client
mcpfy-sdk/widget
mcpfy-sdk/widget-bridge
mcpfy-sdk/auth
```

This allows consumers to import only the functionality they need.

For example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
```

or:

```typescript
import { MCPClient } from "mcpfy-sdk/client";
```

or:

```typescript
import { HostRuntime, useCallTool } from "mcpfy-sdk/widget";
```

## 46. Build Architecture

The package is written in TypeScript and uses:

- tsup
- TypeScript
- Vite
- Vitest

The build process performs two primary operations:

1. Bundles runtime JavaScript using tsup.
2. Generates TypeScript declaration files using tsc.

The build script is:

```
rimraf dist && tsup && tsc --emitDeclarationOnly --declaration
```

The generated package is placed under:

```
dist/
```

The package exports compiled JavaScript and TypeScript declaration files from this directory.

## 47. Dependency Architecture

The SDK intentionally builds on established MCP and web technologies.

Important dependencies include:

- `@modelcontextprotocol/sdk`
- `@modelcontextprotocol/ext-apps`
- `@mcp-ui/server`
- `jose`
- `react`
- `react-dom`
- `vite`
- `tailwindcss`

The official MCP SDK provides the underlying protocol implementation.

jose provides JWT/JWK cryptographic verification.

React and Vite support interactive widget development and bundling.

## 48. Architectural Design Principles

The implementation follows several important design principles.

### 48.1 Thin abstraction

mcpfy simplifies common MCP development without completely hiding the official SDK.

This is demonstrated by:

```
server.nativeServer
```

which exposes the underlying MCP server.

### 48.2 Separation of concerns

Different responsibilities are separated into dedicated modules:

```
Server       → mcp-server.ts
Tools        → tools.ts
Prompts      → prompts.ts
Resources    → resources.ts
Transport    → transport.ts
Auth         → auth/
Widgets      → widgets/
Client       → client/
React runtime → widget-react/
```

This makes the codebase easier to maintain and extend.

### 48.3 Transport independence

Application code does not need to know whether the MCP server is running through stdio or HTTP.

The transport is selected when calling:

```typescript
server.listen(...)
```

### 48.4 Protocol abstraction

Widget code does not need to be tightly coupled to a single host protocol.

The runtime provides common APIs such as:

```typescript
useCallTool()
useHostContext()
useWidgetState()
useModelContext()
useLayoutMode()
useOpenExternal()
useSendFollowUp()
```

while internally adapting to the supported host.

### 48.5 Capability-based behavior

The widget runtime checks host capabilities before using optional functionality.

This allows the same widget implementation to work across hosts with different feature sets.

### 48.6 Provider-independent authentication

JWT verification is implemented generically through issuer, JWKS URI, and optional audience configuration.

Therefore, the authentication architecture is not tied to a single identity provider.

## 49. Overall Request Flow

A typical MCP tool request can be represented as:

```
MCP Client / Host
        │
        ▼
Transport
        │
        ▼
Official MCP Server
        │
        ▼
mcpfy registration layer
        │
        ▼
Tool callback
        │
        ▼
ToolContext
        │
        ▼
Application logic
        │
        ▼
Response helper / structured output
        │
        ▼
Official MCP response
        │
        ▼
Client / Host
```

For a widget-enabled tool, the flow additionally becomes:

```
Tool callback
     │
     ├── structuredContent
     │
     └── widget content
             │
             ▼
        Widget protocol
             │
             ▼
          Host UI
```

## 50. Overall System Architecture

The complete architecture can be summarized as:

```
                           ┌───────────────────────┐
                           │      MCP Host         │
                           │ Claude / App / Client │
                           └───────────┬───────────┘
                                       │
                                       ▼
                              ┌─────────────────┐
                              │    Transport    │
                              │ stdio / HTTP    │
                              └────────┬────────┘
                                       │
                                       ▼
                         ┌──────────────────────────┐
                         │       mcpfy MCPServer    │
                         ├──────────────────────────┤
                         │                          │
                         │  Tools                   │
                         │  Prompts                 │
                         │  Resources               │
                         │  Resource Templates      │
                         │  Authentication          │
                         │  Refresh / Subscriptions │
                         │  Remote MCP Mounting     │
                         │  Widgets                 │
                         │                          │
                         └────────────┬─────────────┘
                                      │
                                      ▼
                         ┌──────────────────────────┐
                         │ Official MCP SDK         │
                         │ @modelcontextprotocol    │
                         └────────────┬─────────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
                    ▼                                   ▼
          ┌──────────────────┐                ┌──────────────────┐
          │ MCP Client Layer │                │ Widget Layer     │
          │                  │                │                  │
          │ MCPClient        │                │ React Runtime    │
          │ Connectors       │                │ Host Adapter     │
          │ Sessions         │                │ Protocol Bridge  │
          └──────────────────┘                └────────┬─────────┘
                                                       │
                                      ┌────────────────┼────────────────┐
                                      ▼                ▼                ▼
                                   MCP-UI          MCP Apps         Apps SDK
```

This architecture allows mcpfy to provide a unified developer experience across MCP server development, client connectivity, authentication, and interactive UI while continuing to rely on the official MCP ecosystem underneath.