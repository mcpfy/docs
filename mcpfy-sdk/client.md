# MCP Client

The `mcpfy` client API provides a simple way to connect to MCP servers and interact with their tools, prompts, and resources.

It is designed around three main concepts:

- **MCPClient** — manages connections to one or more configured MCP servers.
- **MCPSession** — represents an active connection to a specific MCP server.
- **Connectors** — handle the underlying transport, currently supporting **stdio** and **HTTP**.

The client API is intentionally lightweight and forwards MCP operations to the official Model Context Protocol TypeScript SDK. :contentReference[oaicite:0]{index=0}

---

## Installation

```bash
npm install mcpfy-sdk
```

```ts
import { MCPClient } from "mcpfy-sdk/client";
```

Or import all client utilities:

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

## Architecture

```text
MCPClient
   │
   ├── Server Configuration
   │       ├── Stdio configuration
   │       └── HTTP configuration
   │
   ├── createSession()
   │
   ▼
MCPSession
   │
   ▼
BaseConnector
   │
   ├── StdioConnector
   │       └── StdioClientTransport
   │
   └── HttpConnector
           └── StreamableHTTPClientTransport
```

### Responsibilities

| Component | Responsibility |
|------------|----------------|
| `MCPClient` | Manages multiple named MCP server connections |
| `MCPSession` | Provides operations for one connected server |
| `BaseConnector` | Common connection and MCP operation layer |
| `StdioConnector` | Connects through stdio |
| `HttpConnector` | Connects through HTTP |
| `createConnectorFromConfig()` | Selects connector from configuration |

---

# MCPClient

`MCPClient` is the primary entry point.

```ts
import { MCPClient } from "mcpfy-sdk/client";
```

## Configuration

```ts
interface MCPClientConfig {
  mcpServers: Record<string, ServerConfig>;
}
```

### Example

```ts
const client = new MCPClient({
  mcpServers: {
    calculator: {
      command: "node",
      args: ["calculator-server.js"],
    },

    weather: {
      url: "http://localhost:3000/mcp",
    },
  },
});
```

---

## Creating a Session

```ts
const session = await client.createSession("calculator");
```

The method:

1. Checks existing sessions
2. Looks up configuration
3. Creates connector
4. Connects to MCP server
5. Creates `MCPSession`
6. Stores session
7. Returns session

---

## Creating All Sessions

```ts
const sessions = await client.createAllSessions();

const calculator = sessions.calculator;
const weather = sessions.weather;
```

Sessions are created concurrently.

---

## Getting a Session

```ts
const session = client.getSession("calculator");
```

Returns:

```ts
MCPSession | undefined
```

---

## Closing Sessions

```ts
await client.closeAllSessions();
```

Example:

```ts
try {
  const session = await client.createSession("calculator");

  const result = await session.callTool("add", {
    a: 10,
    b: 20,
  });

  console.log(result);
} finally {
  await client.closeAllSessions();
}
```

---

# Server Configuration

```ts
type ServerConfig =
  | {
      command: string;
      args?: string[];
      env?: Record<string, string>;
      cwd?: string;
    }
  | {
      url: string;
      headers?: Record<string, string>;
      authToken?: string;
      authProvider?: OAuthClientProvider;
    };
```

---

# Stdio Server Example

```ts
const client = new MCPClient({
  mcpServers: {
    localServer: {
      command: "node",
      args: ["server.js"],
    },
  },
});
```

### Advanced Example

```ts
const client = new MCPClient({
  mcpServers: {
    database: {
      command: "node",
      args: ["database-server.js"],
      env: {
        DATABASE_URL: "postgresql://localhost/mydb",
      },
      cwd: "./servers",
    },
  },
});
```

---

# HTTP Server Example

```ts
const client = new MCPClient({
  mcpServers: {
    remoteServer: {
      url: "http://localhost:3000/mcp",
    },
  },
});
```

### Custom Headers

```ts
const client = new MCPClient({
  mcpServers: {
    remoteServer: {
      url: "https://example.com/mcp",
      headers: {
        "X-API-Key": "your-api-key",
      },
    },
  },
});
```

### Bearer Authentication

```ts
const client = new MCPClient({
  mcpServers: {
    protectedServer: {
      url: "https://example.com/mcp",
      authToken: "your-access-token",
    },
  },
});
```

Automatically sends:

```http
Authorization: Bearer your-access-token
```

### OAuth

```ts
const client = new MCPClient({
  mcpServers: {
    protectedServer: {
      url: "https://example.com/mcp",
      authProvider: oauthProvider,
    },
  },
});
```

---

# MCPSession

Create a session:

```ts
const session = await client.createSession("calculator");
```

A session provides access to:

- Tools
- Prompts
- Resources
- Resource contents

---

## Tools

### List Tools

```ts
const tools = await session.listTools();

for (const tool of tools) {
  console.log(tool.name);
}
```

### Call Tool

```ts
const result = await session.callTool("add", {
  a: 10,
  b: 20,
});

console.log(result);
```

---

## Prompts

### List Prompts

```ts
const prompts = await session.listPrompts();
```

### Get Prompt

```ts
const result = await session.getPrompt("summarize", {
  topic: "artificial intelligence",
});
```

---

## Resources

### List Resources

```ts
const resources = await session.listResources();
```

### Read Resource

```ts
const result = await session.readResource(
  "file:///example/data.txt"
);

console.log(result);
```

---

## Close Session

```ts
await session.close();
```

---

# Connectors

## StdioConnector

```ts
import { StdioConnector } from "mcpfy-sdk/client";

const connector = new StdioConnector({
  command: "node",
  args: ["server.js"],
});
```

---

## HttpConnector

```ts
import { HttpConnector } from "mcpfy-sdk/client";

const connector = new HttpConnector({
  url: "http://localhost:3000/mcp",
});
```

Supports:

```ts
{
  url: "https://example.com/mcp",
  headers: {
    "X-API-Key": "secret",
  }
}
```

Bearer Auth:

```ts
{
  url: "https://example.com/mcp",
  authToken: "access-token",
}
```

OAuth:

```ts
{
  url: "https://example.com/mcp",
  authProvider: oauthProvider,
}
```

---

## createConnectorFromConfig

```ts
const connector = createConnectorFromConfig({
  command: "node",
  args: ["server.js"],
});
```

Creates a `StdioConnector`.

```ts
const connector = createConnectorFromConfig({
  url: "http://localhost:3000/mcp",
});
```

Creates an `HttpConnector`.

---

# Complete Example

```ts
import { MCPClient } from "mcpfy-sdk/client";

const client = new MCPClient({
  mcpServers: {
    calculator: {
      command: "node",
      args: ["calculator-server.js"],
    },
  },
});

try {
  const session = await client.createSession("calculator");

  const tools = await session.listTools();

  console.log("Available tools:");
  for (const tool of tools) {
    console.log(`- ${tool.name}`);
  }

  const result = await session.callTool("add", {
    a: 10,
    b: 20,
  });

  console.log("Tool result:", result);
} finally {
  await client.closeAllSessions();
}
```

---

# Multiple MCP Servers

```ts
const client = new MCPClient({
  mcpServers: {
    calculator: {
      command: "node",
      args: ["calculator-server.js"],
    },

    weather: {
      url: "http://localhost:3000/mcp",
    },

    database: {
      url: "https://example.com/database/mcp",
      authToken: "access-token",
    },
  },
});
```

```ts
const calculator = await client.createSession("calculator");
const weather = await client.createSession("weather");
const database = await client.createSession("database");
```

Or:

```ts
const sessions = await client.createAllSessions();
```

---

# Error Handling

```ts
try {
  const session = await client.createSession("remote");

  const result = await session.callTool("some_tool", {
    value: "example",
  });

  console.log(result);
} catch (error) {
  console.error("MCP operation failed:", error);
}
```

---

# Public Exports

```ts
MCPClient
MCPSession
BaseConnector
StdioConnector
HttpConnector
createConnectorFromConfig
```

Types:

```ts
MCPClientConfig
ServerConfig
```

---

# Summary

```ts
const client = new MCPClient({
  mcpServers: {
    myServer: {
      url: "http://localhost:3000/mcp",
    },
  },
});

const session = await client.createSession("myServer");

const tools = await session.listTools();

const result = await session.callTool("my_tool", {
  value: "example",
});

await client.closeAllSessions();
```

**Use `MCPClient` for application-level connection management, `MCPSession` for interacting with a connected server, and connector classes when transport-level control is required.**