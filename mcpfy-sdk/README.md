
# mcpfy

Build MCP servers in minutes, not hours.

mcpfy is a lightweight TypeScript SDK for building and consuming Model Context Protocol (MCP) servers and clients.

It provides a clean, declarative API for working with:

- MCP tools
- Prompts
- Resources
- Resource templates
- Interactive UI widgets
- HTTP and stdio transports
- Authentication and JWT/JWKS verification
- MCP client sessions
- Remote MCP server mounting
- React-based widget development
- MCP Apps, MCP-UI, and Apps SDK integrations

mcpfy is designed to simplify common MCP development tasks while remaining close to the official MCP ecosystem.

It wraps the official `@modelcontextprotocol/sdk` rather than replacing it, allowing developers to use mcpfy for convenience while retaining access to the underlying official MCP server implementation when advanced functionality is required.

## Table of Contents

- [mcpfy](#mcpfy)
  - [Table of Contents](#table-of-contents)
  - [What is MCP?](#what-is-mcp)
  - [What is mcpfy?](#what-is-mcpfy)
  - [Why mcpfy?](#why-mcpfy)
    - [Server creation](#server-creation)
    - [Declarative registration](#declarative-registration)
    - [Multiple transports](#multiple-transports)
    - [Authentication](#authentication)
    - [Client abstraction](#client-abstraction)
    - [Interactive widgets](#interactive-widgets)
    - [React runtime](#react-runtime)
  - [Core Features](#core-features)
    - [MCP Server](#mcp-server)
    - [MCP Client](#mcp-client)
    - [Tools](#tools)
    - [Prompts](#prompts)
    - [Resources](#resources)
    - [Resource Templates](#resource-templates)
    - [Widgets](#widgets)
    - [Authentication](#authentication-1)
  - [Architecture](#architecture)
  - [Project Structure](#project-structure)
  - [Requirements](#requirements)
  - [Installation](#installation)
  - [Creating Your First MCP Server](#creating-your-first-mcp-server)
  - [Creating Tools](#creating-tools)
  - [Tool Context](#tool-context)
  - [Prompts](#prompts-1)
  - [Resources](#resources-1)
  - [Resource Templates](#resource-templates-1)
  - [HTTP Transport](#http-transport)
    - [Custom MCP Path](#custom-mcp-path)
    - [Port Configuration](#port-configuration)
  - [Stdio Transport](#stdio-transport)
    - [Listen Result](#listen-result)
  - [Server Configuration](#server-configuration)
    - [name](#name)
    - [version](#version)
    - [description](#description)
    - [basePath](#basepath)
    - [icon](#icon)
    - [auth](#auth)
    - [widgetsDir](#widgetsdir)
  - [Authentication](#authentication-2)
  - [JWT/JWKS Authentication](#jwtjwks-authentication)
    - [Authentication Information](#authentication-information)
      - [Subject](#subject)
      - [Scopes](#scopes)
      - [Claims](#claims)
    - [Authentication Providers](#authentication-providers)
  - [MCP Client](#mcp-client-1)
    - [MCP Sessions](#mcp-sessions)
    - [Creating All Sessions](#creating-all-sessions)
    - [Accessing an Existing Session](#accessing-an-existing-session)
    - [Closing Sessions](#closing-sessions)
  - [Connecting to MCP Servers](#connecting-to-mcp-servers)
  - [Remote MCP Server Mounting](#remote-mcp-server-mounting)
  - [Widgets](#widgets-1)
    - [Registering Widgets](#registering-widgets)
    - [Widget Registration Flow](#widget-registration-flow)
    - [Widget Data](#widget-data)
  - [React Widget Runtime](#react-widget-runtime)
    - [Host Runtime](#host-runtime)
    - [Host Protocol Detection](#host-protocol-detection)
    - [Host Context](#host-context)
    - [Tool Payload](#tool-payload)
    - [Calling Tools from Widgets](#calling-tools-from-widgets)
      - [Generic form](#generic-form)
      - [Named tool form](#named-tool-form)
    - [Linked Tool](#linked-tool)
    - [Sending Follow-Up Messages](#sending-follow-up-messages)
    - [Opening External Links](#opening-external-links)
    - [Layout Modes](#layout-modes)
    - [Themes](#themes)
      - [ThemeProvider](#themeprovider)
  - [Widget State](#widget-state)
    - [View State](#view-state)
  - [Model Context](#model-context)
  - [View Tools](#view-tools)
  - [Host Capabilities](#host-capabilities)
  - [Server Icons](#server-icons)
  - [Refreshing MCP Data](#refreshing-mcp-data)
    - [Refresh a resource](#refresh-a-resource)
    - [Refresh resources](#refresh-resources)
    - [Refresh tools](#refresh-tools)
    - [Refresh prompts](#refresh-prompts)
  - [Resource Subscriptions](#resource-subscriptions)
  - [Error Handling](#error-handling)
  - [Using the Native MCP Server](#using-the-native-mcp-server)
  - [Package Exports](#package-exports)
    - [Main package](#main-package)
    - [Server](#server)
    - [Client](#client)
    - [Widget Bridge](#widget-bridge)
    - [React Widget](#react-widget)
    - [Authentication](#authentication-3)
  - [Development](#development)
  - [Building the Package](#building-the-package)
  - [Testing](#testing)
  - [CLI](#cli)
  - [Dependency Model](#dependency-model)
  - [Design Philosophy](#design-philosophy)
    - [1. Keep MCP accessible](#1-keep-mcp-accessible)
    - [2. Declarative APIs](#2-declarative-apis)
    - [3. Protocol flexibility](#3-protocol-flexibility)
    - [4. Transport flexibility](#4-transport-flexibility)
    - [5. Graceful degradation](#5-graceful-degradation)
    - [6. Structured data first](#6-structured-data-first)
  - [Complete Example](#complete-example)
  - [Lifecycle](#lifecycle)
  - [Summary](#summary)
  - [License](#license)

## What is MCP?

Model Context Protocol (MCP) is an open protocol that allows AI applications to interact with external tools, data sources, and services in a standardized way.

Instead of every AI application implementing its own integration system, MCP provides a common interface between an AI host and external capabilities.

An MCP server can expose things such as:

- Tools
- Resources
- Prompts
- Structured data
- Interactive interfaces

An MCP client connects to these servers and uses the capabilities they expose.

A simplified MCP architecture looks like this:

```
┌─────────────────────┐
│     AI / Host       │
│                     │
│ Claude / ChatGPT /  │
│ Other MCP Host      │
└──────────┬──────────┘
           │
           │ MCP
           ▼
┌─────────────────────┐
│     MCP Client      │
└──────────┬──────────┘
           │
           │
           ▼
┌─────────────────────┐
│     MCP Server      │
│                     │
│  Tools              │
│  Prompts            │
│  Resources          │
│  Widgets            │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ External Systems    │
│ APIs / DB / Files   │
│ Services / Logic    │
└─────────────────────┘
```

mcpfy provides abstractions for both sides of this architecture.

## What is mcpfy?

mcpfy is a TypeScript SDK that makes it easier to create MCP servers and clients.

Instead of working directly with the lower-level MCP SDK APIs for every operation, developers can use a simple API such as:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
});

server.tool(
  {
    name: "get_weather",
    description: "Get the current weather",
    schema: {
      city: "string",
    },
  },
  async ({ city }) => {
    return {
      city,
      temperature: 25,
      condition: "Sunny",
    };
  }
);

await server.listen();
```

The goal is to reduce MCP boilerplate without hiding the underlying protocol.

## Why mcpfy?

Building an MCP server directly with the official SDK is powerful, but applications often need additional infrastructure around it.

mcpfy provides convenient abstractions for common functionality:

### Server creation

A server can be created with a simple configuration object.

```typescript
const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});
```

### Declarative registration

Tools, prompts, resources, and widgets can be registered directly on the server.

```typescript
server
  .tool(...)
  .prompt(...)
  .resource(...);
```

### Multiple transports

mcpfy supports:

- stdio
- HTTP

### Authentication

HTTP MCP servers can be protected using configurable authentication providers.

### Client abstraction

The SDK also provides:

- `MCPClient`

for managing connections and sessions with MCP servers.

### Interactive widgets

mcpfy supports UI integrations across multiple MCP UI protocols.

### React runtime

The widget runtime provides React hooks for interacting with the host environment.

## Core Features

### MCP Server

Create and configure MCP servers using:

- `MCPServer`

Supported server capabilities include:

- Tools
- Prompts
- Resources
- Resource templates
- Widgets
- Authentication
- Server icons
- Remote server mounting
- Resource subscriptions
- Refresh notifications

### MCP Client

The client API provides:

- `MCPClient`
- `MCPSession`

The client supports configured MCP servers and manages reusable sessions.

### Tools

Tools allow an MCP server to expose executable functionality to an MCP client.

Examples include:

- Calling an API
- Searching a database
- Performing calculations
- Creating files
- Running business logic

### Prompts

Prompts allow an MCP server to expose reusable prompt templates to clients.

### Resources

Resources provide read-oriented data to MCP clients.

Resources can represent:

- Files
- Documents
- API data
- Configuration
- Application state

### Resource Templates

Resource templates allow resources to be parameterized.

For example:

```
users/{userId}
```

can represent different user resources.

### Widgets

mcpfy supports interactive UI components associated with MCP tools.

Widgets can work with multiple protocols including:

- MCP-UI
- MCP Apps
- Apps SDK

### Authentication

HTTP MCP servers can require authentication.

mcpfy includes support for bearer-token authentication and JWT/JWKS verification.

## Architecture

The package is organized into several major layers.

```
                         mcpfy-sdk
                            │
          ┌─────────────────┴─────────────────┐
          │                                   │
       Server                              Client
          │                                   │
   ┌──────┼────────┐                   ┌──────┴──────┐
   │      │        │                   │             │
 Tools Prompts Resources           Connectors     Sessions
   │      │        │                   │             │
   └──────┼────────┘                   └──────┬──────┘
          │                                   │
          └──────────── MCP SDK ──────────────┘
                            │
                            ▼
                  @modelcontextprotocol/sdk
```

The server layer provides a high-level API around the official MCP server.

The client layer manages connections and sessions.

The widget layer adds interactive UI capabilities.

The authentication layer handles bearer tokens and JWT-based authentication.

## Project Structure

The main package is organized approximately as follows:

```
src/
│
├── index.ts
│
├── server/
│   ├── mcp-server.ts
│   ├── tools.ts
│   ├── prompts.ts
│   ├── resources.ts
│   ├── context.ts
│   ├── transport.ts
│   ├── refresh.ts
│   ├── subscriptions.ts
│   ├── mount-remote.ts
│   ├── icon.ts
│   │
│   ├── auth/
│   │   ├── middleware.ts
│   │   ├── jwks-verifier.ts
│   │   ├── presets.ts
│   │   ├── types.ts
│   │   ├── middleware.ts
│   │   ├── jwks-verifier.ts
│   │   └── well-known.ts
│   │
│   └── widgets/
│       ├── index.ts
│       ├── attach.ts
│       ├── bundle.ts
│       ├── csp.ts
│       ├── html.ts
│       ├── html-shell.ts
│       ├── prepare.ts
│       ├── registry.ts
│       ├── resolve.ts
│       ├── types.ts
│       └── vite-plugin.ts
│
├── client/
│   ├── index.ts
│   ├── mcp-client.ts
│   ├── session.ts
│   └── connectors.ts
│
├── client-widget/
│   ├── index.ts
│   ├── mcp-apps.ts
│   ├── mcp-ui-actions.ts
│   └── apps-sdk-types.ts
│
├── widget-react/
│   ├── index.ts
│   ├── runtime.tsx
│   └── host-capabilities.ts
│
├── auth/
│   ├── index.ts
│   ├── node-oauth-provider.ts
│   ├── oauth-session-store.ts
│   ├── file-kv-store.ts
│   └── kv-store.ts
│
├── shared/
│   └── response-helpers.ts
│
└── cli/
    └── mcpfy.ts
```

Each directory is responsible for a specific part of the SDK.

## Requirements

mcpfy is a TypeScript/Node.js SDK.

The package specifies the following Node.js versions:

- Node.js 20.19.0+
- or
- Node.js 22.12.0+

A modern TypeScript environment is recommended.

The package uses ES modules:

```json
{
  "type": "module"
}
```

## Installation

Install the package using your preferred package manager.

```bash
npm install mcpfy-sdk
```

or:

```bash
pnpm add mcpfy-sdk
```

or:

```bash
yarn add mcpfy-sdk
```

The package uses zod for schema support.

Install it if your application needs schema validation:

```bash
npm install zod
```

React is an optional peer dependency for applications that use the React widget runtime.

## Creating Your First MCP Server

Import `MCPServer` from the server entry point:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
```

Create a server:

```typescript
const server = new MCPServer({
  name: "example-server",
  version: "1.0.0",
});
```

Register a tool:

```typescript
server.tool(
  {
    name: "hello",
    description: "Say hello",
  },
  async () => {
    return {
      message: "Hello from mcpfy!",
    };
  }
);
```

Start the server:

```typescript
await server.listen();
```

By default, the server uses the stdio transport.

## Creating Tools

Tools are one of the primary ways an MCP server exposes functionality.

The basic structure is:

```typescript
server.tool(definition, callback);
```

For example:

```typescript
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
```

The callback receives the tool input.

A tool can return structured data:

```typescript
return {
  result: 42,
};
```

mcpfy internally registers the tool with the underlying official MCP server.

## Tool Context

Tool callbacks can receive contextual information from the MCP request.

The SDK provides `ToolContext` functionality for operations such as:

- Accessing request context
- Working with authentication information
- Forwarding authentication headers
- Logging
- Sampling
- Asking for URLs where supported

The context can be accessed as the second callback argument:

```typescript
server.tool(
  {
    name: "example",
  },
  async (input, ctx) => {
    // use input
    // use ctx

    return {
      success: true,
    };
  }
);
```

This keeps application logic separate from MCP transport details.

## Prompts

Prompts allow an MCP server to expose reusable prompt definitions.

A prompt can be registered using:

```typescript
server.prompt(...)
```

Example:

```typescript
server.prompt(
  {
    name: "code_review",
    description: "Generate a code review prompt",
  },
  async ({ code }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Review the following code:\n\n${code}`,
          },
        },
      ],
    };
  }
);
```

The exact prompt definition and callback structure are handled by the mcpfy prompt registration layer.

## Resources

Resources allow an MCP server to expose readable data.

Register a resource using:

```typescript
server.resource(...)
```

Example:

```typescript
server.resource(
  {
    name: "config",
    uri: "config://application",
    description: "Application configuration",
  },
  async () => {
    return {
      contents: [
        {
          uri: "config://application",
          text: JSON.stringify({
            environment: "production",
          }),
        },
      ],
    };
  }
);
```

Resources are useful when the client needs to retrieve information rather than execute an operation.

## Resource Templates

For parameterized resources, mcpfy provides:

```typescript
server.resourceTemplate(...)
```

For example, a server can expose resources conceptually like:

```
user://123
user://456
user://789
```

instead of registering every possible resource individually.

This is useful for APIs, databases, and other systems where resource identifiers are dynamic.

## HTTP Transport

mcpfy supports HTTP-based MCP servers.

Start a server using:

```typescript
await server.listen({
  transport: "http",
});
```

By default:

```
host = localhost
port = 3000
path = /mcp
```

The resulting endpoint is therefore:

```
http://localhost:3000/mcp
```

A custom port can be provided:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

A custom host can also be specified:

```typescript
await server.listen({
  transport: "http",
  host: "0.0.0.0",
  port: 4000,
});
```

### Custom MCP Path

The MCP endpoint defaults to:

```
/mcp
```

A custom base path can be configured:

```typescript
const server = new MCPServer({
  name: "weather",
  version: "1.0.0",
  basePath: "/weather",
});
```

The resulting endpoint becomes:

```
http://localhost:3000/weather
```

### Port Configuration

mcpfy resolves the HTTP port using the following priority:

1. `listen({ port })`
2. `--port` command-line argument
3. `PORT` environment variable
4. `3000`

For example:

```bash
node server.js --port 8080
```

or:

```bash
node server.js --port=8080
```

An application can also use:

```
PORT=8080
```

The SDK exposes:

```typescript
parsePortFromArgv()
```

for parsing command-line port arguments.

## Stdio Transport

stdio is the default transport.

Simply call:

```typescript
await server.listen();
```

or explicitly:

```typescript
await server.listen({
  transport: "stdio",
});
```

This transport is particularly useful for MCP hosts that launch an MCP server as a local process.

For example:

```
MCP Host
   │
   │ stdin/stdout
   ▼
mcpfy server process
```

### Listen Result

`listen()` returns a `ListenResult`.

For stdio:

```json
{
  "transport": "stdio"
}
```

For HTTP, the result contains information about the running server:

```typescript
const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(result.url);
```

The HTTP result includes:

```json
{
  "transport": "http",
  "port": 4000,
  "host": "localhost",
  "url": "http://localhost:4000/mcp"
}
```

## Server Configuration

The primary server configuration is:

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

The server name.

```typescript
name: "weather-server"
```

### version

The server version.

```typescript
version: "1.0.0"
```

### description

Optional server description.

```typescript
description: "Provides weather information."
```

### basePath

HTTP endpoint path.

```typescript
basePath: "/weather"
```

### icon

Server icon advertised through MCP server initialization.

It can represent:

- Remote URLs
- data: URIs
- Local file paths
- file: URLs

Local files can be converted into data URIs so MCP clients can display them.

### auth

Authentication configuration for HTTP servers.

### widgetsDir

Root directory used for widget folders.

The default is:

```
src/widgets
```

## Authentication

mcpfy supports authentication for HTTP MCP servers.

Authentication is configured through:

```typescript
auth?: AuthConfig
```

The authentication middleware reads the standard HTTP header:

```
Authorization: Bearer <token>
```

The flow is:

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
     ├── valid ──► authenticated request
     │
     └── invalid ► rejected request
```

Requests without a valid bearer token are rejected.

## JWT/JWKS Authentication

mcpfy provides a generic JWT verifier through:

```typescript
jwksVerifier()
```

Import it using:

```typescript
import {
  MCPServer,
  jwksVerifier,
} from "mcpfy-sdk/server";
```

Example:

```typescript
const verifyToken = jwksVerifier({
  issuer: "https://issuer.example.com",
  jwksUri: "https://issuer.example.com/.well-known/jwks.json",
  audience: "my-api",
});
```

The verifier uses jose and a remote JWKS endpoint.

It validates JWTs using:

- Issuer
- Audience, when configured
- Signature
- Standard JWT verification rules

The JWKS key set is managed through jose's remote JWKS functionality, including its caching and key rotation behavior.

### Authentication Information

A successful authentication produces `AuthInfo`.

Conceptually:

```json
{
  "sub": "...",
  "scopes": "...",
  "claims": "...",
  "token": "..."
}
```

The JWT verifier extracts:

#### Subject

```
sub
```

#### Scopes

If the JWT contains:

```
scope: "read write admin"
```

mcpfy converts it into:

```json
["read", "write", "admin"]
```

#### Claims

The complete JWT payload is exposed through:

```
claims
```

### Authentication Providers

The server package exports authentication helpers including:

- `jwksVerifier`
- `oauthAuth0Provider`
- `oauthWorkOSProvider`

These allow applications to use common OAuth/OIDC authentication setups without implementing the complete provider integration themselves.

## MCP Client

mcpfy also provides a client abstraction.

Import:

```typescript
import { MCPClient } from "mcpfy-sdk/client";
```

Create a client:

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

### MCP Sessions

A session can be created using:

```typescript
const session = await client.createSession("weather");
```

If the session already exists, mcpfy reuses the existing session.

This means repeated calls to:

```typescript
client.createSession("weather");
```

do not unnecessarily create multiple connections.

### Creating All Sessions

When an application needs to connect to all configured servers:

```typescript
const sessions = await client.createAllSessions();
```

This creates sessions for all configured MCP servers.

The result is a record keyed by server name:

```json
{
  "weather": "weatherSession",
  "database": "databaseSession",
  "search": "searchSession"
}
```

### Accessing an Existing Session

An existing session can be retrieved with:

```typescript
const session = client.getSession("weather");
```

If no session has been created:

```
undefined
```

is returned.

### Closing Sessions

Applications should close their sessions when they are no longer required.

```typescript
await client.closeAllSessions();
```

This closes all managed sessions and clears the internal session map.

## Connecting to MCP Servers

The client layer supports connectors.

The package exposes:

- `BaseConnector`
- `StdioConnector`
- `HttpConnector`
- `createConnectorFromConfig`

Connectors abstract the actual MCP transport.

Conceptually:

```
MCPClient
    │
    ▼
MCPSession
    │
    ▼
Connector
    │
    ├── StdioConnector
    │
    └── HttpConnector
```

This separates session management from transport-specific connection logic.

## Remote MCP Server Mounting

mcpfy can expose tools, prompts, and resources from another HTTP MCP server through the current server.

Use:

```typescript
server.mountRemote(...)
```

Example:

```typescript
await server.mountRemote({
  weather: {
    // remote server configuration
  },
});
```

Remote tools and prompts are namespaced using:

```
{alias}__{original}
```

For example:

```
weather__get_forecast
```

This makes it possible to compose multiple MCP servers into a single MCP endpoint.

Conceptually:

```
                Main MCP Server
                      │
          ┌───────────┴───────────┐
          │                       │
      Local Tools          Mounted Servers
                                  │
                        ┌─────────┴─────────┐
                        │                   │
                   Weather MCP        Database MCP
```

## Widgets

Widgets provide an interactive UI layer on top of MCP functionality.

mcpfy supports multiple UI protocols through a unified abstraction.

The supported protocol types include:

- mcp-ui
- mcp-apps
- apps-sdk

The goal is to allow widget developers to build a single UI experience while mcpfy handles protocol-specific integration.

### Registering Widgets

Widgets can be associated with tools.

The server exposes:

```typescript
server.tool(...)
```

with widget-related configuration.

The SDK also retains:

```typescript
server.widget(...)
```

for backwards compatibility.

The `widget()` method is currently marked as deprecated in favor of passing widget configuration directly through `server.tool()`.

### Widget Registration Flow

A widget registration roughly follows this process:

```
Widget Definition
       │
       ▼
registerWidget()
       │
       ▼
attachWidgetProtocols()
       │
       ├── MCP-UI
       ├── MCP Apps
       └── Apps SDK
       │
       ▼
Register MCP Tool
       │
       ▼
Tool Result
       │
       ├── Text / structured data
       │
       └── UI content when supported
```

This allows a tool call to produce both structured application data and UI content.

### Widget Data

A widget callback can produce structured data.

The registered tool returns:

```json
{
  "content": [],
  "structuredContent": "data"
}
```

The structured output is useful because the host can consume the data independently of the visual representation.

This separation is important:

```
Tool Data
   │
   ├── Model / Application
   │
   └── Widget UI
```

## React Widget Runtime

mcpfy provides a React runtime through:

```
mcpfy-sdk/widget
```

The runtime provides React components and hooks for interacting with the MCP host.

The central component is:

```jsx
<HostRuntime>
  ...
</HostRuntime>
```

Example:

```jsx
import { HostRuntime } from "mcpfy-sdk/widget";

export default function App() {
  return (
    <HostRuntime toolName="weather">
      <WeatherWidget />
    </HostRuntime>
  );
}
```

### Host Runtime

`HostRuntime` establishes the runtime environment used by widget hooks.

It handles:

- Host connection
- Protocol detection
- Tool input
- Tool output
- Theme
- Layout mode
- Host capabilities
- Widget state
- Tool calls
- Follow-up messages
- External links
- Model context
- View tools

### Host Protocol Detection

Widgets may run in different environments.

mcpfy exposes:

```typescript
useHostProtocol()
```

which identifies the current host protocol.

Possible values include:

- apps-sdk
- mcp-apps
- mcp-ui
- none

The runtime adapts its behavior based on the detected host.

### Host Context

Use:

```typescript
useHostContext()
```

to access host information.

Example:

```typescript
const host = useHostContext();

console.log(host.protocol);
console.log(host.layoutMode);
console.log(host.locale);
console.log(host.platform);
```

The context includes:

```json
{
  "protocol": "...",
  "layoutMode": "...",
  "locale": "...",
  "platform": "...",
  "capabilities": "..."
}
```

### Tool Payload

The current tool input/output can be accessed using:

```typescript
useToolPayload()
```

Example:

```typescript
const payload = useToolPayload();

if (payload.isPending) {
  return <div>Loading...</div>;
}

return <pre>{JSON.stringify(payload.output, null, 2)}</pre>;
```

The payload contains:

```json
{
  "input": "...",
  "output": "...",
  "isPending": "...",
  "error": "..."
}
```

### Calling Tools from Widgets

Use:

```typescript
useCallTool()
```

There are two supported forms.

#### Generic form

```typescript
const callTool = useCallTool();

await callTool("get_weather", {
  city: "Delhi",
});
```

#### Named tool form

```typescript
const weather = useCallTool("get_weather");

await weather.call({
  city: "Delhi",
});
```

The named version provides:

```json
{
  "call": "...",
  "isPending": "...",
  "data": "...",
  "error": "..."
}
```

This allows widgets to manage tool execution state easily.

### Linked Tool

A widget can access the tool it is associated with using:

```typescript
useLinkedTool()
```

Example:

```typescript
const tool = useLinkedTool();

console.log(tool.name);

await tool.call({
  city: "Delhi",
});
```

This avoids hard-coding the linked tool name inside the component.

### Sending Follow-Up Messages

Widgets can send follow-up messages to the host using:

```typescript
useSendFollowUp()
```

Example:

```typescript
const sendFollowUp = useSendFollowUp();

await sendFollowUp("Show me tomorrow's forecast.");
```

Depending on the host, mcpfy routes the request through the appropriate host API.

### Opening External Links

Use:

```typescript
useOpenExternal()
```

Example:

```typescript
const openExternal = useOpenExternal();

openExternal("https://example.com");
```

The runtime selects the appropriate host-specific mechanism.

### Layout Modes

Widgets can request different display modes using:

```typescript
useLayoutMode()
```

Example:

```typescript
const layout = useLayoutMode();

console.log(layout.mode);
console.log(layout.available);

await layout.request("fullscreen");
```

The runtime supports host-provided display modes while also handling environments where the host does not provide a display-mode API.

### Themes

mcpfy supports:

- light
- dark

The runtime initially detects the browser's preferred color scheme.

The SDK provides:

```typescript
useHostTheme()
```

to access the current theme.

#### ThemeProvider

Applications can also use:

```jsx
<ThemeProvider>
  ...
</ThemeProvider>
```

The provider tracks system theme changes and exposes theme state to widgets.

The runtime also sets:

```
data-mcpfy-theme
```

on the document root.

## Widget State

Widgets can maintain state that can persist through supported hosts.

Use:

```typescript
useWidgetState()
```

Example:

```typescript
const { state, setState } = useWidgetState();

await setState({
  selectedCity: "Delhi",
});
```

The runtime updates local state and, where supported, forwards it to the host.

### View State

For local UI state combined with host persistence, use:

```typescript
useViewState()
```

Example:

```typescript
const [state, setState] = useViewState({
  selectedTab: "overview",
});
```

State updates are:

- Stored locally.
- Sent to the host when supported.
- Published as model context when supported.

This provides a convenient abstraction for stateful widgets.

## Model Context

Widgets can publish structured information back to the model/host.

Use:

```typescript
useModelContext()
```

Example:

```typescript
const { supported, publish } = useModelContext();

if (supported) {
  await publish({
    text: "The user selected Delhi.",
    structuredContent: {
      city: "Delhi",
    },
  });
}
```

This allows UI interactions to communicate meaningful state back to the model.

## View Tools

MCP Apps hosts can support tools registered directly by a mounted view.

mcpfy provides:

```typescript
useViewTool()
```

Example:

```typescript
useViewTool(
  {
    name: "refresh_data",
    description: "Refresh the displayed data",
  },
  async () => {
    return {
      refreshed: true,
    };
  }
);
```

The runtime registers the tool with the host when the required capability is available.

In hosts that do not support view tools, the registration becomes a no-op.

## Host Capabilities

Widgets can inspect host capabilities through:

```typescript
useHostContext()
```

or the runtime's capability system.

Capabilities can describe support for features such as:

- Display modes
- Model context
- View tools
- Host-specific interactions

This allows widgets to degrade gracefully when running in different environments.

## Server Icons

An MCP server can advertise an icon.

Example:

```typescript
const server = new MCPServer({
  name: "weather",
  version: "1.0.0",
  icon: "./icon.png",
});
```

Supported icon sources include:

- Remote URL
- data: URI
- Local file
- file: URL

Local icons are converted into data URIs so they can be advertised through MCP initialization.

## Refreshing MCP Data

mcpfy provides methods for notifying subscribed clients about changes.

### Refresh a resource

```typescript
await server.refreshResource("resource://example");
```

This tells subscribed clients that the resource has new content.

### Refresh resources

```typescript
server.refreshResources();
```

This tells clients to list resources again.

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

## Resource Subscriptions

mcpfy enables resource subscriptions on the underlying MCP server.

This allows clients to subscribe to resource updates and receive notifications when content changes.

The server automatically enables:

```
resources.subscribe
```

and:

```
resources.listChanged
```

capabilities.

## Error Handling

Tool callbacks can throw normal JavaScript errors.

For example:

```typescript
server.tool(
  {
    name: "divide",
  },
  async ({ a, b }) => {
    if (b === 0) {
      throw new Error("Cannot divide by zero");
    }

    return {
      result: a / b,
    };
  }
);
```

The underlying MCP layer is responsible for converting the failure into the appropriate MCP response.

Widget runtime errors are exposed through:

```
error
```

in the tool payload and tool handles.

## Using the Native MCP Server

mcpfy does not prevent access to the official MCP implementation.

The `MCPServer` class exposes:

```
nativeServer
```

Example:

```typescript
const server = new MCPServer({
  name: "advanced-server",
  version: "1.0.0",
});

server.nativeServer;
```

The type is based on:

```
McpServer
```

from:

```
@modelcontextprotocol/sdk/server/mcp.js
```

This is an intentional escape hatch.

Developers can use mcpfy's high-level API for most functionality and access the native server when they need lower-level or advanced MCP SDK features.

## Package Exports

mcpfy provides several entry points.

### Main package

```typescript
import ... from "mcpfy-sdk";
```

The root package provides the main SDK exports.

### Server

```typescript
import {
  MCPServer,
  jwksVerifier,
} from "mcpfy-sdk/server";
```

The server entry point exposes:

- `MCPServer`
- Tool types
- Prompt types
- Resource types
- Widget types
- Authentication types
- Authentication providers
- Response helpers
- Server icons
- Remote server configuration

### Client

```typescript
import {
  MCPClient,
  MCPSession,
  HttpConnector,
  StdioConnector,
} from "mcpfy-sdk/client";
```

The client entry point exposes:

- `MCPClient`
- `MCPSession`
- `BaseConnector`
- `StdioConnector`
- `HttpConnector`
- `createConnectorFromConfig`

### Widget Bridge

The package provides:

```
mcpfy-sdk/widget-bridge
```

for client-side widget/host integration.

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

### Authentication

Authentication helpers are available through:

```typescript
import {
  NodeOAuthClientProvider,
  ensureAuthorized,
  OAuthSessionStore,
  FileKVStore,
} from "mcpfy-sdk/auth";
```

## Development

The repository uses TypeScript and is structured as a package within the TypeScript workspace.

The package source is located at:

```
typescript/packages/mcpfy
```

Install dependencies from the project workspace before building.

## Building the Package

The package provides a build script:

```bash
npm run build
```

The build process:

1. Removes the previous dist directory.
2. Builds the package using tsup.
3. Generates TypeScript declaration files.

The resulting compiled package is placed under:

```
dist/
```

The package exposes compiled JavaScript and TypeScript declaration files from this directory.

## Testing

The package uses Vitest.

Run the test suite with:

```bash
npm test
```

For watch mode:

```bash
npm run test:watch
```

The package therefore supports both:

```
vitest run
```

and:

```
vitest
```

through the provided npm scripts.

## CLI

The package also provides the mcpfy CLI executable.

The package configuration exposes:

```json
{
  "bin": {
    "mcpfy": "./dist/src/cli/mcpfy.js"
  }
}
```

After building/installing the package, the CLI can be used through:

```bash
mcpfy
```

The CLI provides an entry point for command-line workflows around mcpfy.

## Dependency Model

mcpfy is built on top of the official MCP ecosystem.

Important dependencies include:

- `@modelcontextprotocol/sdk`
- `@modelcontextprotocol/ext-apps`
- `@mcp-ui/server`
- `jose`
- `react`
- `react-dom`
- `vite`
- `tailwindcss`

The most important architectural dependency is:

```
@modelcontextprotocol/sdk
```

mcpfy uses the official SDK as its underlying MCP implementation.

## Design Philosophy

mcpfy follows several important design principles.

### 1. Keep MCP accessible

mcpfy simplifies MCP development without attempting to hide the MCP SDK completely.

The underlying native server remains accessible:

```
server.nativeServer
```

### 2. Declarative APIs

Common MCP operations should require minimal boilerplate.

Instead of manually wiring every MCP registration step:

```typescript
server.tool(...)
server.prompt(...)
server.resource(...)
```

can be used directly.

### 3. Protocol flexibility

Widgets may need to work across different hosts and UI protocols.

mcpfy therefore provides a common runtime while adapting to:

- MCP-UI
- MCP Apps
- Apps SDK

### 4. Transport flexibility

The same server implementation can run through:

```
stdio
```

or:

```
HTTP
```

without changing the application's tool logic.

### 5. Graceful degradation

Widget features are capability-driven.

If a host does not support a particular capability, the runtime can fall back or perform a no-op rather than requiring every widget to implement host-specific logic.

### 6. Structured data first

Widgets and tools can expose structured output independently from their UI.

This allows:

```
Tool
 │
 ├── Structured output
 │
 └── Interactive UI
```

rather than tying application data directly to presentation.

## Complete Example

The following example demonstrates the basic server structure.

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "example-server",
  version: "1.0.0",
  description: "Example MCP server built with mcpfy",
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

server.prompt(
  {
    name: "explain",
    description: "Explain a concept",
  },
  async ({ topic }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Explain ${topic} clearly.`,
          },
        },
      ],
    };
  }
);

await server.listen({
  transport: "stdio",
});
```

For an HTTP server:

```typescript
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "http-example",
  version: "1.0.0",
});

server.tool(
  {
    name: "hello",
    description: "Return a greeting",
  },
  async ({ name }) => {
    return {
      message: `Hello, ${name}!`,
    };
  }
);

const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(`MCP server running at ${result.url}`);
```

The endpoint will be:

```
http://localhost:4000/mcp
```

## Lifecycle

A typical mcpfy server lifecycle looks like:

```
Create MCPServer
       │
       ▼
Configure server
       │
       ▼
Register tools/prompts/resources/widgets
       │
       ▼
Prepare widgets
       │
       ▼
Start transport
       │
       ├───────────────┐
       │               │
      stdio           HTTP
       │               │
       └───────┬───────┘
               ▼
          Handle MCP
           requests
               │
               ▼
        Refresh / Updates
               │
               ▼
             close()
```

When the server is no longer needed:

```typescript
await server.close();
```

This closes:

- HTTP transport, when active
- Mounted remote connections
- Native MCP server resources

## Summary

mcpfy provides a higher-level developer experience for the Model Context Protocol while maintaining compatibility with the official MCP SDK.

Its main building blocks are:

- `MCPServer`
- `MCPClient`
- `MCPSession`
- Tools
- Prompts
- Resources
- Resource Templates
- Widgets
- React Runtime
- Authentication
- Connectors
- Transports

A typical application can therefore follow a simple architecture:

```
                ┌───────────────────┐
                │      AI Host      │
                └─────────┬─────────┘
                          │
                          │ MCP
                          ▼
                ┌───────────────────┐
                │   mcpfy Client    │
                │                   │
                │ MCPClient         │
                │ MCPSession        │
                │ Connectors        │
                └─────────┬─────────┘
                          │
                          │ MCP
                          ▼
                ┌───────────────────┐
                │   mcpfy Server    │
                │                   │
                │ Tools             │
                │ Prompts           │
                │ Resources         │
                │ Widgets           │
                │ Authentication    │
                └─────────┬─────────┘
                          │
              ┌───────────┼───────────┐
              ▼           ▼           ▼
            APIs        Database     Services
```

The result is a compact SDK that simplifies MCP development while preserving access to the official MCP implementation underneath.

## License

mcpfy is released under the MIT License.
