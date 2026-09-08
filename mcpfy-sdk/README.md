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

---

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
    - [npm](#npm)
    - [pnpm](#pnpm)
    - [yarn](#yarn)
- [Creating Your First MCP Server](#creating-your-first-mcp-server)
- [Creating Tools](#creating-tools)
  - [Tool Response Helpers](#tool-response-helpers)
    - [`text()`](#text)
    - [`markdown()`](#markdown)
    - [`object()`](#object)
- [Tool Context](#tool-context)
  - [Logging](#logging)
  - [Forwarding Authentication Headers](#forwarding-authentication-headers)
- [Prompts](#prompts-1)
- [Resources](#resources-1)
- [Resource Templates](#resource-templates-1)
- [HTTP Transport](#http-transport)
  - [Custom MCP Path](#custom-mcp-path)
  - [Port Configuration](#port-configuration)
  - [Host Configuration](#host-configuration)
  - [Silent Startup](#silent-startup)
  - [Listen Result](#listen-result)
- [Stdio Transport](#stdio-transport)
- [Server Configuration](#server-configuration)
  - [`name`](#name)
  - [`version`](#version)
  - [`description`](#description)
  - [`basePath`](#basepath)
  - [`icon`](#icon)
  - [`auth`](#auth)
  - [`widgetsDir`](#widgetsdir)
- [Authentication](#authentication-2)
- [JWT/JWKS Authentication](#jwtjwks-authentication)
  - [Authentication Information](#authentication-information)
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
  - [Widget Directory Structure](#widget-directory-structure)
  - [Widget Content](#widget-content)
  - [Widget Size](#widget-size)
  - [Widget Build](#widget-build)
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
- [OAuth Client Helpers](#oauth-client-helpers)
- [Development](#development)
- [Building the Package](#building-the-package)
- [Testing](#testing)
- [CLI](#cli)
  - [`mcpfy dev`](#mcpfy-dev)
  - [`mcpfy build`](#mcpfy-build)
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
- [Quick Start Summary](#quick-start-summary)
- [Summary](#summary)
- [License](#license)

---

## What is MCP?

Model Context Protocol (MCP) is an open protocol that allows AI applications to interact with external tools, data sources, and services in a standardized way.

Instead of every AI application implementing its own integration system, MCP provides a common interface between an AI host and external capabilities.

An MCP server can expose:

- Tools
- Resources
- Prompts
- Structured data
- Interactive interfaces

An MCP client connects to these servers and uses the capabilities they expose.

A simplified MCP architecture looks like this:

```text
┌─────────────────────┐
│      AI / Host      │
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
           ▼
┌─────────────────────┐
│     MCP Server      │
│                     │
│ Tools               │
│ Prompts             │
│ Resources           │
│ Widgets             │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ External Systems    │
│                     │
│ APIs / DB / Files   │
│ Services / Logic    │
└─────────────────────┘
````

mcpfy provides abstractions for both sides of this architecture.

---

## What is mcpfy?

mcpfy is a TypeScript SDK that makes it easier to create MCP servers and clients.

Instead of working directly with lower-level MCP SDK APIs for every operation, developers can use a declarative API.

For example:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
});

server.tool(
  {
    name: "get_weather",
    description: "Get the current weather",
    schema: z.object({
      city: z.string(),
    }),
  },
  async ({ city }) => {
    return object({
      city,
      temperature: 25,
      condition: "Sunny",
    });
  }
);

await server.listen();
```

The goal is to reduce MCP boilerplate without hiding the underlying protocol.

---

## Why mcpfy?

Building an MCP server directly with the official SDK is powerful, but applications often need additional infrastructure around it.

mcpfy provides convenient abstractions for common functionality.

### Server creation

A server can be created with a simple configuration object:

```typescript
const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});
```

### Declarative registration

Tools, prompts, resources, and widgets can be registered directly on the server:

```typescript
server.tool(...);
server.prompt(...);
server.resource(...);
```

### Multiple transports

mcpfy supports:

* stdio
* HTTP

### Authentication

HTTP MCP servers can be protected using configurable authentication providers.

### Client abstraction

The SDK provides:

* `MCPClient`
* `MCPSession`
* `BaseConnector`
* `StdioConnector`
* `HttpConnector`

for managing connections and sessions with MCP servers.

### Interactive widgets

mcpfy supports UI integrations across multiple MCP UI protocols.

### React runtime

The widget runtime provides React components and hooks for interacting with the host environment.

---

# Core Features

## MCP Server

Create and configure MCP servers using:

* `MCPServer`

Supported server capabilities include:

* Tools
* Prompts
* Resources
* Resource templates
* Widgets
* Authentication
* Server icons
* Remote server mounting
* Resource subscriptions
* Refresh notifications

## MCP Client

The client API provides:

* `MCPClient`
* `MCPSession`

The client supports configured MCP servers and manages reusable sessions.

## Tools

Tools allow an MCP server to expose executable functionality to an MCP client.

Examples include:

* Calling an API
* Searching a database
* Performing calculations
* Creating files
* Running business logic

## Prompts

Prompts allow an MCP server to expose reusable prompt templates to clients.

## Resources

Resources provide read-oriented data to MCP clients.

Resources can represent:

* Files
* Documents
* API data
* Configuration
* Application state

## Resource Templates

Resource templates allow resources to be parameterized.

For example:

```text
users/{userId}
```

can represent different user resources.

This is useful for APIs, databases, and other systems where resource identifiers are dynamic.

## Widgets

mcpfy supports interactive UI components associated with MCP tools.

Widgets can work with multiple protocols including:

* MCP-UI
* MCP Apps
* Apps SDK

## Authentication

HTTP MCP servers can require authentication.

mcpfy includes support for bearer-token authentication and JWT/JWKS verification.

---

# Architecture

The package is organized into several major layers:

```text
                         mcpfy-sdk
                             │
             ┌───────────────┴───────────────┐
             │                               │
          Server                           Client
             │                               │
     ┌───────┼────────┐              ┌───────┴───────┐
     │       │        │              │               │
   Tools  Prompts  Resources     Connectors       Sessions
     │       │        │              │               │
     └───────┼────────┘              └───────┬───────┘
             │                               │
             └──────────── MCP SDK ──────────┘
                             │
                             ▼
                  @modelcontextprotocol/sdk
```

The server layer provides a high-level API around the official MCP server.

The client layer manages connections and sessions.

The widget layer adds interactive UI capabilities.

The authentication layer handles bearer tokens and JWT-based authentication.

---

# Project Structure

The main package is organized approximately as follows:

```text
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

---

# Requirements

mcpfy is a TypeScript/Node.js SDK.

The package supports:

* Node.js `^20.19.0`
* Node.js `>=22.12.0`

Node.js 21.x is not included in the supported engine range.

A modern TypeScript environment is recommended.

The package uses ES modules.

Add the following to `package.json`:

```json
{
  "type": "module"
}
```

---

# Installation

Install the package using your preferred package manager.

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

When using Zod schemas, install `zod` explicitly as a peer dependency.

If your application uses Zod schemas, install Zod explicitly:

```bash
npm install zod
```

For React widget development, install the React dependencies required by your application.

---

# Creating Your First MCP Server

For a new project, the recommended quick-start command is:

```bash
npx create-mcpfy-app@latest
```

Alternatively, create a TypeScript project manually.

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
import { MCPServer, text } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "example-server",
  version: "1.0.0",
});

server.tool(
  {
    name: "hello",
    description: "Say hello",
  },
  async () => {
    return text("Hello from mcpfy!");
  }
);

await server.listen();
```

By default, the server uses the stdio transport.

---

# Creating Tools

Tools are one of the primary ways an MCP server exposes functionality.

The basic structure is:

```typescript
server.tool(definition, callback);
```

A tool schema is defined using Zod:

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
  async ({ a, b }) => {
    return object({
      result: a + b,
    });
  }
);
```

The callback receives validated tool input.

## Tool Response Helpers

mcpfy provides response helpers for tool callbacks.

### `text()`

```typescript
import { text } from "mcpfy-sdk/server";

return text("Hello!");
```

### `markdown()`

```typescript
import { markdown } from "mcpfy-sdk/server";

return markdown("# Hello\n\nThis is Markdown.");
```

### `object()`

Use `object()` for structured tool output:

```typescript
import { object } from "mcpfy-sdk/server";

return object({
  result: 42,
  success: true,
});
```

Tool callbacks should not return arbitrary bare objects such as:

```typescript
return {
  result: 42,
};
```

Use the appropriate response helper instead.

---

# Tool Context

Tool callbacks can receive contextual information from the MCP request.

The context can be accessed as the second callback argument:

```typescript
server.tool(
  {
    name: "example",
    description: "Example tool",
  },
  async (input, ctx) => {
    // use input
    // use ctx

    return object({
      success: true,
    });
  }
);
```

The context provides functionality such as:

* Accessing request context
* Working with authentication information
* Forwarding authentication headers
* Logging
* Sampling
* Asking for URLs where supported

## Logging

Use a log level when calling `ctx.log()`:

```typescript
ctx.log("info", "Processing request");
```

The signature is:

```typescript
ctx.log(level, message);
```

## Forwarding Authentication Headers

mcpfy provides helpers for safely forwarding supported authentication headers to upstream services.

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

The SDK also exports:

```typescript
FORWARDABLE_AUTH_HEADER_NAMES
```

Applications should not blindly forward arbitrary inbound headers.

---

# Prompts

Prompts allow an MCP server to expose reusable prompt definitions.

A prompt can be registered using:

```typescript
server.prompt(...)
```

Prompt arguments are described using a Zod schema:

```typescript
import { z } from "zod";

server.prompt(
  {
    name: "code_review",
    description: "Generate a code review prompt",
    schema: z.object({
      code: z.string(),
    }),
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

The prompt definition uses `schema` for its arguments.

---

# Resources

Resources allow an MCP server to expose readable data.

Register a resource using:

```typescript
server.resource(...)
```

Example:

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

Resources are useful when the client needs to retrieve information rather than execute an operation.

---

# Resource Templates

For parameterized resources, mcpfy provides:

```typescript
server.resourceTemplate(...)
```

The callback receives:

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

This allows a server to expose resources such as:

```text
user://123
user://456
user://789
```

without registering every possible resource individually.

---

# HTTP Transport

mcpfy supports HTTP-based MCP servers.

Start a server using:

```typescript
await server.listen({
  transport: "http",
});
```

By default:

```text
host = localhost
port = 3000
path = /mcp
```

The resulting endpoint is:

```text
http://localhost:3000/mcp
```

## Custom MCP Path

The MCP endpoint defaults to:

```text
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

```text
http://localhost:3000/weather
```

## Port Configuration

mcpfy resolves the HTTP port using this priority:

1. `listen({ port })`
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
node server.js --port 8080
```

and:

```text
PORT=8080
```

Both command-line formats are supported:

```bash
node server.js --port 8080
```

and:

```bash
node server.js --port=8080
```

The SDK exposes:

```typescript
parsePortFromArgv()
```

for parsing command-line port arguments.

## Host Configuration

A custom host can be specified:

```typescript
await server.listen({
  transport: "http",
  host: "0.0.0.0",
  port: 4000,
});
```

## Silent Startup

Suppress the HTTP startup message with:

```typescript
await server.listen({
  transport: "http",
  silent: true,
});
```

## Listen Result

For HTTP:

```typescript
const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(result.url);
```

The result contains information about the running server:

```json
{
  "transport": "http",
  "port": 4000,
  "host": "localhost",
  "url": "http://localhost:4000/mcp"
}
```

---

# Stdio Transport

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

Conceptually:

```text
MCP Host
   │
   │ stdin/stdout
   ▼
mcpfy server process
```

For stdio, the listen result is:

```json
{
  "transport": "stdio"
}
```

---

# Server Configuration

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

## `name`

The server name.

```typescript
name: "weather-server"
```

## `version`

The server version.

```typescript
version: "1.0.0"
```

## `description`

Optional server description.

```typescript
description: "Provides weather information."
```

## `basePath`

HTTP endpoint path.

```typescript
basePath: "/weather"
```

The default is:

```text
/mcp
```

## `icon`

Server icon advertised through MCP server initialization.

Supported icon sources include:

* Remote URLs
* `data:` URIs
* Local file paths
* `file:` URLs

Local files can be converted into data URIs so MCP clients can display them.

## `auth`

Authentication configuration for HTTP servers.

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

## `widgetsDir`

Root directory used for widget folders.

The default is:

```text
src/widgets
```

---

# Authentication

mcpfy supports authentication for HTTP MCP servers.

Authentication is configured through:

```typescript
auth?: AuthConfig
```

The authentication middleware reads the standard HTTP header:

```text
Authorization: Bearer <token>
```

The flow is:

```text
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

OAuth authentication also enables the MCP protected-resource metadata endpoint:

```text
/.well-known/oauth-protected-resource
```

---

# JWT/JWKS Authentication

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

Create a verifier:

```typescript
const issuer = "https://auth.example.com";

const verifyToken = jwksVerifier({
  issuer,
  jwksUri: `${issuer}/.well-known/jwks.json`,
  audience: "my-mcp-server",
});
```

Configure it on the server:

```typescript
const server = new MCPServer({
  name: "protected-server",
  version: "1.0.0",
  auth: {
    type: "oauth",
    verifyToken,
    authorizationServers: [issuer],
  },
});
```

The authentication type is:

```typescript
type: "oauth"
```

There is no:

```typescript
type: "jwt"
```

configuration.

The verifier validates JWTs using:

* Issuer
* Audience, when configured
* Signature
* Standard JWT verification rules

The JWKS key set is managed through the underlying JOSE remote JWKS functionality.

## Authentication Information

A successful authentication produces authentication information containing values such as:

```text
sub
scopes
claims
token
```

If the JWT contains:

```text
scope: "read write admin"
```

mcpfy exposes the scopes as:

```json
[
  "read",
  "write",
  "admin"
]
```

The complete JWT payload is available through:

```text
claims
```

## Authentication Providers

The server package exports authentication helpers including:

* `jwksVerifier`
* `oauthAuth0Provider`
* `oauthWorkOSProvider`

---

# MCP Client

mcpfy also provides a client abstraction.

Import:

```typescript
import { MCPClient } from "mcpfy-sdk/client";
```

Create a client with configured MCP servers:

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

## MCP Sessions

A session can be created using:

```typescript
const session = await client.createSession("weather");
```

If the session already exists, mcpfy reuses the existing session.

## Creating All Sessions

When an application needs to connect to all configured servers:

```typescript
const sessions = await client.createAllSessions();
```

This creates sessions for all configured MCP servers.

## Accessing an Existing Session

An existing session can be retrieved with:

```typescript
const session = client.getSession("weather");
```

If no session has been created, `undefined` is returned.

## Closing Sessions

Applications should close their sessions when they are no longer required:

```typescript
await client.closeAllSessions();
```

---

# Connecting to MCP Servers

The client layer supports connectors.

The package exposes:

* `BaseConnector`
* `StdioConnector`
* `HttpConnector`
* `createConnectorFromConfig`

Connectors abstract the actual MCP transport.

Conceptually:

```text
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

---

# Remote MCP Server Mounting

mcpfy can expose tools, prompts, and resources from another MCP server through the current server.

Use:

```typescript
server.mountRemote(...)
```

Remote server configurations can be mounted and exposed through the current server.

Remote tools and prompts are namespaced using:

```text
{alias}__{original}
```

For example:

```text
weather__get_forecast
```

This makes it possible to compose multiple MCP servers into a single MCP endpoint.

Conceptually:

```text
                Main MCP Server
                       │
          ┌────────────┴────────────┐
          │                         │
      Local Tools             Mounted Servers
                                      │
                           ┌──────────┴──────────┐
                           │                     │
                     Weather MCP          Database MCP
```

---

# Widgets

Widgets provide an interactive UI layer on top of MCP functionality.

mcpfy supports multiple UI protocols through a unified abstraction.

Supported protocol types include:

* `mcp-ui`
* `mcp-apps`
* `apps-sdk`

The goal is to allow widget developers to build a single UI experience while mcpfy handles protocol-specific integration.

## Registering Widgets

Widgets can be associated with tools.

For example:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
});

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
      temperature: 25,
    });
  }
);
```

The widget name corresponds to the widget directory:

```text
src/widgets/weather/
```

The standard widget entry point is:

```text
src/widgets/weather/main.tsx
```

## Widget Directory Structure

A basic widget project can use:

```text
src/
└── widgets/
    └── weather/
        └── main.tsx
```

Create the widget entry point at:

```text
src/widgets/weather/main.tsx
```

The standard widget runtime convention automatically handles the required runtime integration for this entry point.

Do not manually add another `ThemeProvider` or `HostRuntime` around the standard widget entry point when the SDK is already injecting the runtime.

## Widget Content

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

## Widget Build

During development:

```bash
mcpfy dev
```

Build widgets for production with:

```bash
mcpfy build
```

A production server that depends on built widgets should have the widget assets built before starting the server.

---

# React Widget Runtime

mcpfy provides a React runtime through:

```text
mcpfy-sdk/widget
```

The runtime provides React components and hooks for interacting with the MCP host.

The central runtime component is:

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

> For the standard `src/widgets/<name>/main.tsx` widget convention, the SDK handles the required runtime setup automatically. Follow the standard widget entry-point convention instead of manually double-wrapping the runtime.

## Host Runtime

`HostRuntime` establishes the runtime environment used by widget hooks.

It handles:

* Host connection
* Protocol detection
* Tool input
* Tool output
* Theme
* Layout mode
* Host capabilities
* Widget state
* Tool calls
* Follow-up messages
* External links
* Model context
* View tools

## Host Protocol Detection

Widgets may run in different environments.

mcpfy exposes:

```typescript
useHostProtocol()
```

Possible host protocols include:

* `apps-sdk`
* `mcp-apps`
* `mcp-ui`
* `none`

The runtime adapts its behavior based on the detected host.

## Host Context

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

## Tool Payload

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

return (
  <pre>
    {JSON.stringify(payload.output, null, 2)}
  </pre>
);
```

The payload contains information such as:

```text
input
output
isPending
error
```

## Calling Tools from Widgets

Use:

```typescript
useCallTool()
```

### Generic form

```typescript
const callTool = useCallTool();

await callTool("get_weather", {
  city: "Delhi",
});
```

### Named tool form

```typescript
const weather = useCallTool("get_weather");

await weather.call({
  city: "Delhi",
});
```

The named version provides tool execution state such as:

```text
call
isPending
data
error
```

## Linked Tool

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

## Sending Follow-Up Messages

Widgets can send follow-up messages to the host using:

```typescript
useSendFollowUp()
```

Example:

```typescript
const sendFollowUp = useSendFollowUp();

await sendFollowUp("Show me tomorrow's forecast.");
```

## Opening External Links

Use:

```typescript
useOpenExternal()
```

Example:

```typescript
const openExternal = useOpenExternal();

openExternal("https://example.com");
```

## Layout Modes

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

## Themes

mcpfy supports:

* light
* dark

The runtime initially detects the browser's preferred color scheme.

Use:

```typescript
useHostTheme()
```

to access the current theme.

### ThemeProvider

The SDK also provides:

```jsx
<ThemeProvider>
  ...
</ThemeProvider>
```

For the standard widget entry-point convention, do not manually add another provider if the SDK runtime already supplies it.

---

# Widget State

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

## View State

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

State updates can be:

* Stored locally
* Sent to the host when supported
* Published as model context when supported

---

# Model Context

Widgets can publish structured information back to the model or host.

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

---

# View Tools

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

---

# Host Capabilities

Widgets can inspect host capabilities through:

```typescript
useHostContext()
```

or the runtime's capability system.

Capabilities can describe support for features such as:

* Display modes
* Model context
* View tools
* Host-specific interactions

This allows widgets to degrade gracefully when running in different environments.

---

# Server Icons

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

* Remote URL
* `data:` URI
* Local file
* `file:` URL

Local icons are converted into data URIs so they can be advertised through MCP initialization.

---

# Refreshing MCP Data

mcpfy provides methods for notifying clients about changes.

## Refresh a resource

```typescript
await server.refreshResource("resource://example");
```

This tells subscribed clients that the resource has new content.

## Refresh resources

```typescript
server.refreshResources();
```

This tells clients to list resources again.

## Refresh tools

```typescript
server.refreshTools();
```

This tells clients to refresh their tool list.

## Refresh prompts

```typescript
server.refreshPrompts();
```

This tells clients to refresh their prompt list.

---

# Resource Subscriptions

mcpfy enables resource subscriptions on the underlying MCP server.

This allows clients to subscribe to resource updates and receive notifications when content changes.

The server supports resource subscription and resource-list-change capabilities including:

```text
resources.subscribe
resources.listChanged
```

A typical workflow is:

```text
Client subscribes to resource
          │
          ▼
Resource content changes
          │
          ▼
server.refreshResource(uri)
          │
          ▼
Client receives update
```

---

# Error Handling

Tool callbacks can throw normal JavaScript errors.

Example:

```typescript
server.tool(
  {
    name: "divide",
    description: "Divide two numbers",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  },
  async ({ a, b }) => {
    if (b === 0) {
      throw new Error("Cannot divide by zero");
    }

    return object({
      result: a / b,
    });
  }
);
```

The underlying MCP layer is responsible for converting failures into the appropriate MCP response.

Widget runtime errors are exposed through:

```text
error
```

in the tool payload and tool handles.

---

# Using the Native MCP Server

mcpfy does not prevent access to the official MCP implementation.

The `MCPServer` class exposes:

```text
nativeServer
```

Example:

```typescript
const server = new MCPServer({
  name: "advanced-server",
  version: "1.0.0",
});

const nativeServer = server.nativeServer;
```

The native server is based on the official MCP SDK implementation.

This is an intentional escape hatch.

Developers can use mcpfy's high-level API for most functionality and access the native server when they need lower-level or advanced MCP SDK features.

---

# Package Exports

mcpfy provides several entry points.

## Main package

```typescript
import ... from "mcpfy-sdk";
```

The root package provides the main SDK exports.

## Server

```typescript
import {
  MCPServer,
  jwksVerifier,
  object,
  text,
  markdown,
} from "mcpfy-sdk/server";
```

The server entry point exposes functionality for:

* `MCPServer`
* Tool types
* Prompt types
* Resource types
* Widget types
* Authentication types
* Authentication providers
* Response helpers
* Server icons
* Remote server configuration
* Refresh functionality
* Authentication header forwarding

## Client

```typescript
import {
  MCPClient,
  MCPSession,
  HttpConnector,
  StdioConnector,
} from "mcpfy-sdk/client";
```

The client entry point exposes:

* `MCPClient`
* `MCPSession`
* `BaseConnector`
* `StdioConnector`
* `HttpConnector`
* `createConnectorFromConfig`

## Widget Bridge

The package provides:

```text
mcpfy-sdk/widget-bridge
```

for client-side widget and host integration.

The widget bridge includes APIs such as:

```text
postIntent
postNotify
connectMcpApps
App
PostMessageTransport
```

## React Widget

React widget functionality is exposed through:

```typescript
import {
  HostRuntime,
  useCallTool,
  useHostContext,
  useToolPayload,
} from "mcpfy-sdk/widget";
```

## Authentication

Authentication helpers are available through:

```typescript
import {
  NodeOAuthClientProvider,
  ensureAuthorized,
  OAuthSessionStore,
  FileKVStore,
} from "mcpfy-sdk/auth";
```

---

# OAuth Client Helpers

For Node.js OAuth clients, create the provider through its static `create()` method.

```typescript
const provider = await NodeOAuthClientProvider.create({
  // provider options
});
```

The constructor is private and should not be called directly.

Use `ensureAuthorized()` with both the provider and the target server URL:

```typescript
const serverUrl = "https://example.com/mcp";

await ensureAuthorized(
  provider,
  serverUrl
);
```

---

# Development

The repository uses TypeScript and is structured as a package within the TypeScript workspace.

The package source is located at:

```text
typescript/packages/mcpfy
```

Install dependencies from the project workspace before building.

---

# Building the Package

The package provides a build script:

```bash
npm run build
```

The build process:

1. Removes the previous `dist` directory.
2. Builds the package using tsup.
3. Generates TypeScript declaration files.

The resulting compiled package is placed under:

```text
dist/
```

The package exposes compiled JavaScript and TypeScript declaration files from this directory.

---

# Testing

The package uses Vitest.

Run the test suite with:

```bash
npm test
```

For watch mode:

```bash
npm run test:watch
```

The underlying test commands are:

```text
vitest run
```

and:

```text
vitest
```

---

# CLI

The package provides the `mcpfy` CLI executable.

The package configuration exposes:

```json
{
  "bin": {
    "mcpfy": "./dist/src/cli/mcpfy.js"
  }
}
```

After building or installing the package, the CLI can be used through:

```bash
mcpfy
```

## `mcpfy dev`

Starts the widget development workflow:

```bash
mcpfy dev
```

## `mcpfy build`

Builds widget assets for production:

```bash
mcpfy build
```

Build widget assets before starting a production server that depends on those widgets.

---

# Dependency Model

mcpfy is built on top of the official MCP ecosystem.

Important dependencies include:

* `@modelcontextprotocol/sdk`
* `@modelcontextprotocol/ext-apps`
* `@mcp-ui/server`
* `jose`
* `react`
* `react-dom`
* `vite`
* `tailwindcss`

The most important architectural dependency is:

```text
@modelcontextprotocol/sdk
```

mcpfy uses the official SDK as its underlying MCP implementation.

---

# Design Philosophy

mcpfy follows several important design principles.

## 1. Keep MCP accessible

mcpfy simplifies MCP development without attempting to hide the MCP SDK completely.

The underlying native server remains accessible:

```typescript
server.nativeServer
```

## 2. Declarative APIs

Common MCP operations should require minimal boilerplate.

Instead of manually wiring every MCP registration step:

```typescript
server.tool(...)
server.prompt(...)
server.resource(...)
```

can be used directly.

## 3. Protocol flexibility

Widgets may need to work across different hosts and UI protocols.

mcpfy therefore provides a common runtime while adapting to:

* MCP-UI
* MCP Apps
* Apps SDK

## 4. Transport flexibility

The same server implementation can run through:

```text
stdio
```

or:

```text
HTTP
```

without changing the application's tool logic.

## 5. Graceful degradation

Widget features are capability-driven.

If a host does not support a particular capability, the runtime can fall back or perform a no-op rather than requiring every widget to implement host-specific logic.

## 6. Structured data first

Widgets and tools can expose structured output independently from their UI.

This allows:

```text
Tool
 │
 ├── Structured output
 │
 └── Interactive UI
```

rather than tying application data directly to presentation.

---

# Complete Example

The following example demonstrates a basic HTTP MCP server:

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

const result = await server.listen({
  transport: "http",
  port: 4000,
});

console.log(`MCP server running at ${result.url}`);
```

The server exposes:

* Two executable tools
* One MCP resource
* An HTTP MCP endpoint
* Structured and text tool responses

The endpoint is:

```text
http://localhost:4000/mcp
```

---

# Lifecycle

A typical mcpfy server lifecycle looks like:

```text
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
Build widgets when required
       │
       ▼
Start transport
       │
       ├───────────────┐
       │               │
     stdio            HTTP
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

* HTTP transport, when active
* Mounted remote connections
* Native MCP server resources

---

# Quick Start Summary

For a new application:

```bash
npx create-mcpfy-app@latest
```

Or install the SDK manually:

```bash
npm install mcpfy-sdk zod
```

Create a server:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "my-server",
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
    return object({
      result: a + b,
    });
  }
);

await server.listen({
  transport: "http",
});
```

For widget development:

```bash
mcpfy dev
```

For a production widget build:

```bash
mcpfy build
```

---

# Summary

mcpfy provides a higher-level developer experience for the Model Context Protocol while maintaining compatibility with the official MCP SDK.

Its main building blocks are:

* `MCPServer`
* `MCPClient`
* `MCPSession`
* Tools
* Prompts
* Resources
* Resource Templates
* Widgets
* React Runtime
* Authentication
* Connectors
* Transports
* Resource subscriptions
* Refresh notifications

A typical application can therefore follow a simple architecture:

```text
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
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           APIs        Database     Services
```

The result is a compact SDK that simplifies MCP development while preserving access to the official MCP implementation underneath.

---

# License

mcpfy is released under the MIT License.
