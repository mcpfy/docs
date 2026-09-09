
# Tools

Tools allow an MCP server to expose executable functionality to MCP clients.

A tool has a name, description, input schema, and callback. The callback receives validated input and returns an MCP-compatible result.

## Basic Tool

Import `MCPServer` and the response helpers from `mcpfy-sdk/server`, and use Zod to define the input schema.

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

await server.listen();
````

`zod` is a peer dependency of the SDK; install it explicitly in applications that define Zod schemas.

The `schema` property must be a Zod schema. For object-shaped input, use `z.object(...)`.

The callback receives input that has already been validated against the schema.

## Tool Definition

A tool definition can contain the following commonly used properties:

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";
import { z } from "zod";

server.tool(
  {
    name: "weather",
    description: "Get the current weather",
    schema: z.object({
      city: z.string(),
    }),
  },
  async ({ city }) => {
    return text(`Weather information for ${city}`);
  }
);
```

### `name`

The unique name of the tool.

```typescript
name: "weather"
```

### `description`

A human-readable description of what the tool does.

```typescript
description: "Get the current weather"
```

A useful description helps MCP clients and models understand when the tool should be used.

### `schema`

The input schema for the tool.

```typescript
schema: z.object({
  city: z.string(),
})
```

mcpfy uses Zod schemas for tool input validation.

For example:

```typescript
schema: z.object({
  name: z.string(),
  age: z.number().optional(),
  active: z.boolean().default(true),
})
```

## Tool Results

Tool callbacks must return an MCP-compatible result.

mcpfy provides response helpers that create the required MCP content structure.

### Text Results

Use `text()` when the tool should return plain text.

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "example",
  version: "1.0.0",
});

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

await server.listen();
```

### Structured Results

Use `object()` when the tool should return structured data.

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

server.tool(
  {
    name: "calculate",
    description: "Perform a calculation",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  },
  async ({ a, b }) => {
    return object({
      sum: a + b,
      product: a * b,
    });
  }
);

await server.listen();
```

The important distinction is that a tool callback should not return an arbitrary object such as:

```typescript
return {
  result: a + b,
};
```

Instead, wrap structured data with `object()`:

```typescript
return object({
  result: a + b,
});
```

This produces the MCP-compatible response content expected by the SDK.

## Markdown Results

When a response is intended to contain Markdown content, use `markdown()`.

```typescript
import { MCPServer, markdown } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "documentation",
  version: "1.0.0",
});

server.tool(
  {
    name: "get-documentation",
    description: "Return documentation",
    schema: z.object({
      topic: z.string(),
    }),
  },
  async ({ topic }) => {
    return markdown(`# ${topic}\n\nDocumentation for **${topic}**.`);
  }
);

await server.listen();
```

## Optional Input

Zod can be used to define optional values.

```typescript
import { text } from "mcpfy-sdk/server";
import { z } from "zod";

server.tool(
  {
    name: "greet",
    description: "Generate a greeting",
    schema: z.object({
      name: z.string(),
      formal: z.boolean().optional(),
    }),
  },
  async ({ name, formal }) => {
    return text(
      formal
        ? `Good day, ${name}.`
        : `Hey ${name}!`
    );
  }
);
```

## Tool Validation

Because the schema is defined with Zod, invalid input is rejected according to the schema before the tool callback receives it.

For example:

```typescript
schema: z.object({
  a: z.number(),
  b: z.number(),
})
```

expects both `a` and `b` to be numbers.

A string such as:

```json
{
  "a": "10",
  "b": 20
}
```

does not satisfy the schema.

## Multiple Tools

An MCP server can register multiple tools.

```typescript
import { MCPServer, object, text } from "mcpfy-sdk/server";
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

server.tool(
  {
    name: "multiply",
    description: "Multiply two numbers",
    schema: z.object({
      a: z.number(),
      b: z.number(),
    }),
  },
  async ({ a, b }) => {
    return object({
      result: a * b,
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

await server.listen();
```

## Tool Context

Tool callbacks can use the context provided by mcpfy for operations such as logging and interacting with the MCP runtime.

A tool callback can receive the context as an additional argument:

```typescript
server.tool(
  {
    name: "example",
    description: "Example tool",
    schema: z.object({
      value: z.string(),
    }),
  },
  async ({ value }, ctx) => {
    ctx.log("info", `Processing ${value}`);

    return text(`Processed: ${value}`);
  }
);
```

The log method requires a log level followed by the message:

```typescript
ctx.log("info", "Processing request");
```

Do not use:

```typescript
ctx.log("Processing request");
```

## Returning Errors

Tool implementations can throw errors when an operation cannot be completed.

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

Use meaningful error messages so that clients can understand what went wrong.

## Images

mcpfy supports image content in tool responses.

When returning image content, the image data must be supplied as base64-encoded data rather than as a remote URL.

For example, fetch the image, convert it to base64, and pass that data to `image()`:

```typescript
import { image } from "mcpfy-sdk/server";

async function loadImage() {
  const response = await fetch("https://example.com/image.png");
  const buffer = Buffer.from(await response.arrayBuffer());
  const base64 = buffer.toString("base64");
  return image(base64, "image/png");
}
```

Return the result of `loadImage()` from the tool callback.

Do not pass the URL directly where base64 image data is required.

## Client Configuration for Tools

Tools can be exposed by an HTTP MCP server and consumed by an MCP client.

For example, an HTTP server can be started with:

```typescript
await server.listen({
  transport: "http",
  port: 4000,
});
```

The MCP endpoint is:

```text
http://localhost:4000/mcp
```

A client can connect using the server URL:

```typescript
import { MCPClient } from "mcpfy-sdk/client";

const client = new MCPClient({
  mcpServers: {
    remote: { url: "http://localhost:4000/mcp" },
  },
});

const session = await client.createSession("remote");
```

For a stdio server, configure the client with the command used to start the server:

```typescript
import { MCPClient } from "mcpfy-sdk/client";

const client = new MCPClient({
  mcpServers: {
    local: { command: "node", args: ["dist/server.js"] },
  },
});

const session = await client.createSession("local");
```

The transport is inferred from the server configuration. There is no `transport: "stdio"` property in the client `ServerConfig`.

## Typed Tool Calls

mcpfy supports typed tool calls when schemas and types are available.

Defining a Zod schema provides the foundation for strongly typed tool input:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const schema = z.object({
  city: z.string(),
  units: z.enum(["celsius", "fahrenheit"]),
});
```

The callback input is inferred from the schema:

```typescript
server.tool(
  {
    name: "weather",
    description: "Get weather information",
    schema,
  },
  async ({ city, units }) => {
    return object({
      city,
      units,
    });
  }
);
```

This reduces the need for manual input type declarations and keeps runtime validation aligned with TypeScript usage.

## Organizing Tools

For larger projects, tools can be separated into individual modules.

Example:

```text
src/
├── server.ts
└── tools/
    ├── calculator.ts
    └── weather.ts
```

A tool module can export a registration function:

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

export function registerCalculator(server: MCPServer) {
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
}
```

The main server can then register the tool:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
import { registerCalculator } from "./tools/calculator.js";

const server = new MCPServer({
  name: "calculator",
  version: "1.0.0",
});

registerCalculator(server);

await server.listen();
```

## Forwarding Authentication Headers

When an HTTP MCP server needs to make authenticated upstream requests using authentication received from the MCP request, use the SDK's supported authentication-header forwarding helpers.

The relevant APIs include:

```typescript
forwardAuthHeaders
extractForwardableAuthHeaders
FORWARDABLE_AUTH_HEADER_NAMES
```

These helpers provide the supported mechanism for forwarding permitted inbound authentication headers to upstream requests.

Authentication configuration and token verification are covered in [Authentication](authentication.md).

## Tools with Widgets

Tools can be associated with MCP Apps widgets when the server needs to return an interactive UI.

A tool can specify a widget using the SDK's widget configuration.

The widget itself must follow the project's widget directory and build conventions.

See [Widgets](widgets.md) for the complete widget workflow.

## Best Practices

### Use descriptive names

Prefer:

```typescript
name: "get-weather"
```

over:

```typescript
name: "tool1"
```

### Write useful descriptions

A description should explain what the tool does and when it should be used.

```typescript
description: "Get the current weather for a specified city"
```

### Validate all structured input

Use Zod schemas instead of accepting unvalidated objects:

```typescript
schema: z.object({
  city: z.string(),
  country: z.string().optional(),
})
```

### Return SDK-compatible results

Use the response helpers:

```typescript
return text("Done");
```

or:

```typescript
return object({
  success: true,
});
```

rather than returning an arbitrary object.

### Keep tools focused

A tool should generally perform one well-defined operation. Smaller, focused tools are easier for MCP clients and models to understand and use correctly.

## Complete Example

The following example combines schema validation, multiple tools, and MCP-compatible responses:

```typescript
import { MCPServer, object, text } from "mcpfy-sdk/server";
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

await server.listen({
  transport: "http",
  port: 4000,
});
```

The server exposes two tools:

* `add` — returns structured data using `object()`
* `greet` — returns text using `text()`

The examples in this guide use the actual mcpfy tool schema and response patterns so they can be checked against the SDK rather than relying on arbitrary MCP-shaped objects.

