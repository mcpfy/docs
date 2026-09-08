# Tools

Tools are the primary way an MCP server exposes executable functionality to MCP clients.

The MCPfy SDK provides a declarative `server.tool()` API that wraps the official MCP server tool registration while providing convenient support for schemas, callbacks, context, structured results, and widgets.

## Basic Tool

A tool is registered with `server.tool()` by providing a definition and a callback.

```ts
import { MCPServer, text } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "my-server",
  version: "1.0.0",
});

server.tool(
  {
    name: "greet",
    description: "Greet a user",
    schema: {
      name: "string",
    },
  },
  async ({ name }) => {
    return text(`Hello, ${name}!`);
  }
);

await server.listen();
```

The tool definition describes the tool that MCP clients can discover, while the callback contains the logic executed when the tool is called.

## Tool Definition

The `ToolDefinition` type supports the following main properties:

```ts
interface ToolDefinition<TInput = Record<string, any>, TOutput = Record<string, unknown>> {
  name: string;
  title?: string;
  description?: string;
  schema?: unknown;
  widget?: string | WidgetOptions;
  cb?: ToolCallback<TInput, TOutput>;
}
```

### `name`

The unique name of the tool.

```ts
{
  name: "get_weather"
}
```

The name is used by MCP clients when calling the tool.

### `title`

An optional human-readable title for the tool.

```ts
{
  name: "get_weather",
  title: "Get Weather"
}
```

### `description`

A description of what the tool does.

```ts
{
  name: "get_weather",
  description: "Get the current weather for a city"
}
```

Descriptions are useful because MCP clients and models can use them to understand when a tool should be called.

### `schema`

The optional input schema describes the arguments accepted by the tool.

MCPfy supports schema definitions through the SDK's tool registration layer, including Zod-based schemas where appropriate.

For example:

```ts
import { z } from "zod";

server.tool(
  {
    name: "get_weather",
    description: "Get weather information",
    schema: z.object({
      city: z.string(),
      units: z.enum(["celsius", "fahrenheit"]).optional(),
    }),
  },
  async ({ city, units }) => {
    // ...
  }
);
```

The schema is used to describe and validate the tool's input according to the underlying MCP SDK behavior.

## Tool Callback

The callback is executed when the tool is called.

```ts
server.tool(
  {
    name: "add",
    description: "Add two numbers",
    schema: {
      a: "number",
      b: "number",
    },
  },
  async ({ a, b }) => {
    return {
      result: a + b,
    };
  }
);
```

The callback receives the parsed input and can return a tool result.

Callbacks can also access the MCPfy tool context.

```ts
server.tool(
  {
    name: "example",
    description: "Example tool",
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

## Defining the Callback Inside the Definition

Instead of passing the callback as the second argument, it can be supplied through the tool definition when supported by the definition.

```ts
server.tool({
  name: "greet",
  description: "Greet a user",
  schema: {
    name: "string",
  },
  cb: async ({ name }) => {
    return text(`Hello, ${name}!`);
  },
});
```

Passing the callback separately is also supported:

```ts
server.tool(
  {
    name: "greet",
    description: "Greet a user",
    schema: {
      name: "string",
    },
  },
  async ({ name }) => {
    return text(`Hello, ${name}!`);
  }
);
```

## Returning Results

MCPfy provides response helpers for common tool-result formats.

They are exported from `mcpfy-sdk/server`:

```ts
import {
  text,
  markdown,
  image,
  object,
  error,
} from "mcpfy-sdk/server";
```

### Text

Return plain text:

```ts
return text("Hello from MCPfy");
```

### Markdown

Return Markdown content:

```ts
return markdown(`
# Weather

The current temperature is **24°C**.
`);
```

### Image

Return an image result:

```ts
return image("https://example.com/weather.png");
```

### Object

Return structured data:

```ts
return object({
  city: "Delhi",
  temperatureC: 24,
  condition: "Sunny",
});
```

This is useful when the caller needs machine-readable structured output.

### Error

Return an MCP tool error:

```ts
return error("Unable to retrieve weather data");
```

## Structured Output

Tools can return structured data that can be consumed programmatically.

For example:

```ts
server.tool(
  {
    name: "get_weather",
    description: "Get weather information",
  },
  async ({ city }) => {
    return {
      city,
      temperatureC: 24,
      condition: "Sunny",
    };
  }
);
```

MCPfy preserves structured results when registering tools and when interacting with them through its client and widget APIs.

For widget-enabled tools, the result can also be exposed through `structuredContent`.

## Tool Context

Tool callbacks can receive a `ToolContext` as their second argument.

```ts
server.tool(
  {
    name: "example",
    description: "Example tool",
  },
  async (input, ctx) => {
    // Tool implementation
    return text("Done");
  }
);
```

The context provides access to functionality associated with the current MCP request.

MCPfy exposes context-related types through:

```ts
import type {
  ToolContext,
  SampleOptions,
  LogLevel,
  AskUrlOptions,
} from "mcpfy-sdk/server";
```

The context layer also includes helpers for working with authentication headers when a server forwards authenticated requests.

## Authentication-Aware Tools

When authentication is configured for an HTTP MCP server, tool execution can work with the authenticated request context.

Authentication is configured at the server level:

```ts
const server = new MCPServer({
  name: "secure-server",
  version: "1.0.0",
  auth: {
    // authentication configuration
  },
});
```

Tool callbacks can then use the request context provided by MCPfy where appropriate.

For complete authentication configuration, see the [Authentication](./authentication) guide.

## Tools With Widgets

MCPfy allows a tool to be associated with an interactive widget.

For example:

```ts
server.tool(
  {
    name: "weather",
    description: "Get weather information",
    schema: {
      city: "string",
    },
    widget: "weather",
  },
  async ({ city }) => {
    return {
      city,
      temperatureC: 24,
      condition: "Sunny",
    };
  }
);
```

The widget is associated with the tool and can present the tool's result through an interactive UI.

MCPfy supports widget protocols including:

* MCP-UI
* MCP Apps
* Apps SDK

The SDK handles protocol-specific metadata and content generation while keeping the tool registration API consistent.

See [Widgets](./widgets) for the complete widget documentation.

## Tool Registration Flow

When `server.tool()` is called, MCPfy registers the tool with the underlying official MCP server.

Conceptually, the flow is:

```text
server.tool(...)
      │
      ▼
MCPfy tool registration
      │
      ▼
Official MCP Server
      │
      ▼
MCP client discovers the tool
      │
      ▼
Client calls the tool
      │
      ▼
MCPfy callback executes
      │
      ▼
Tool result returned
```

MCPfy therefore provides a higher-level developer API without replacing the underlying MCP implementation.

## Tool Discovery Updates

MCPfy enables the MCP server's `tools/list_changed` capability.

When the available tools change, the server can notify clients that they should refresh their tool list.

You can explicitly request a refresh with:

```ts
server.refreshTools();
```

This is useful for applications where the available tools can change dynamically.

## Calling Tools From a Client

Tools exposed by an MCPfy server can be consumed using `MCPClient`.

```ts
import { MCPClient } from "mcpfy-sdk/client";

const client = new MCPClient({
  mcpServers: {
    weather: {
      transport: "stdio",
      command: "node",
      args: ["weather-server.js"],
    },
  },
});

const session = await client.createSession("weather");

const result = await session.callTool("get_weather", {
  city: "Delhi",
});
```

The client API is documented in [Client](./client).

## TypeScript Types

MCPfy's tool API is generic, allowing input and output types to be represented at compile time.

```ts
server.tool<
  { city: string },
  { city: string; temperatureC: number }
>(
  {
    name: "weather",
    description: "Get weather information",
  },
  async ({ city }) => {
    return {
      city,
      temperatureC: 24,
    };
  }
);
```

This is particularly useful when building larger MCP servers where multiple tools share strongly typed data structures.

## Tool Naming

Use clear, stable names for tools.

Good examples:

```text
get_weather
search_documents
create_ticket
get_customer
send_email
```

Avoid unnecessarily ambiguous names such as:

```text
data
run
doThing
process
```

A descriptive name combined with a useful description makes the tool easier for MCP clients and models to understand.

## Best Practices

### Keep tools focused

A tool should generally perform one clear operation.

Prefer:

```text
search_users
get_user
update_user
```

over a single tool that attempts to perform unrelated operations.

### Write useful descriptions

Tool descriptions should explain what the tool does and when it should be used.

```ts
{
  name: "search_documents",
  description: "Search indexed documents using a text query and return matching documents."
}
```

### Define input schemas

Schemas make tool inputs explicit and improve validation and interoperability.

```ts
schema: z.object({
  query: z.string(),
  limit: z.number().optional(),
})
```

### Return structured data when appropriate

If the result will be consumed programmatically, prefer structured output over encoding everything into a text string.

```ts
return {
  id: "123",
  status: "active",
};
```

### Handle errors explicitly

Use the provided error helper when a tool operation cannot be completed:

```ts
return error("Document could not be found");
```

### Keep callbacks asynchronous when necessary

Tool callbacks can perform asynchronous operations such as API requests or database queries:

```ts
server.tool(
  {
    name: "get_user",
    description: "Get a user by ID",
  },
  async ({ id }) => {
    const user = await database.users.findById(id);

    if (!user) {
      return error("User not found");
    }

    return object(user);
  }
);
```

## API Summary

| API                     | Purpose                                 |
| ----------------------- | --------------------------------------- |
| `server.tool()`         | Register an MCP tool                    |
| `ToolDefinition`        | Describe a tool                         |
| `ToolCallback`          | Implement tool behavior                 |
| `text()`                | Return text content                     |
| `markdown()`            | Return Markdown content                 |
| `image()`               | Return image content                    |
| `object()`              | Return structured data                  |
| `error()`               | Return a tool error                     |
| `server.refreshTools()` | Notify clients to refresh the tool list |

## Related Documentation

* [Getting Started](./getting-started)
* [Architecture](./architecture)
* [Server](./server)
* [Client](./client)
* [Prompts](./prompts)
* [Resources](./resources)
* [Widgets](./widgets)
* [Widget React](./widget-react)
* [Authentication](./authentication)
* [API Reference](./api-reference)