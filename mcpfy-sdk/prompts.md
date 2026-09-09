# MCP Prompts

mcpfy provides a declarative API for defining and registering **MCP prompts**.

Prompts allow an MCP server to expose reusable prompt templates that MCP clients can discover and request with structured arguments.

mcpfy simplifies prompt registration while keeping the underlying MCP prompt model intact.

---

## 1. Overview

A prompt in mcpfy consists of:

* A unique **name**
* An optional **title**
* An optional **description**
* An optional **Zod schema** for prompt arguments
* A **callback** that generates the prompt result

The basic structure is:

```typescript
server.prompt(
  {
    name: "summarize",
    description: "Create a summary of a topic",
    schema: z.object({
      topic: z.string(),
    }),
  },
  async (params, ctx) => {
    // Generate prompt
  }
);
```

The prompt is then available to MCP clients through the standard MCP prompt operations.

---

# 2. Importing Prompt APIs

The prompt types are available from:

```typescript
import {
  MCPServer,
  type PromptDefinition,
  type PromptCallback,
} from "mcpfy-sdk/server";
```

For schemas, use Zod:

```typescript
import { z } from "zod";
```

---

# 3. Registering a Prompt

Prompts are registered using:

```typescript
server.prompt(...)
```

Example:

```typescript
import { MCPServer } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "prompt-server",
  version: "1.0.0",
});

server.prompt(
  {
    name: "summarize",
    description: "Create a summary prompt",
    schema: z.object({
      topic: z.string(),
    }),
  },
  async ({ topic }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Summarize the following topic: ${topic}`,
          },
        },
      ],
    };
  }
);

await server.listen();
```

---

# 4. Prompt Definition

The prompt definition has the following structure:

```typescript
interface PromptDefinition<TInput = Record<string, any>> {
  name: string;
  title?: string;
  description?: string;
  schema?: z.ZodObject<any>;
  cb?: PromptCallback<TInput>;
}
```

### Properties

| Property      | Type             | Required | Description                            |
| ------------- | ---------------- | -------: | -------------------------------------- |
| `name`        | `string`         |      Yes | Unique prompt name                     |
| `title`       | `string`         |       No | Human-readable prompt title            |
| `description` | `string`         |       No | Description of what the prompt does    |
| `schema`      | `z.ZodObject`    |       No | Zod schema describing prompt arguments |
| `cb`          | `PromptCallback` |       No | Prompt handler callback                |

A callback must be supplied either through the second argument of `.prompt()` or through the `cb` property.

---

# 5. Prompt Callback

A prompt callback receives:

```typescript
(params, ctx)
```

where:

* `params` contains the validated prompt arguments.
* `ctx` is the mcpfy `ToolContext`.

The callback type is:

```typescript
type PromptCallback<TInput = Record<string, any>> = (
  params: TInput,
  ctx: ToolContext
) => Promise<
  GetPromptResult |
  TypedCallToolResult<any> |
  ToolContentResult
>;
```

This means prompt callbacks can return either a standard MCP `GetPromptResult` or the same simplified content-helper results supported by tools.

---

# 6. Using a Zod Schema

A Zod object can be used to describe prompt arguments.

```typescript
schema: z.object({
  topic: z.string(),
})
```

For multiple arguments:

```typescript
schema: z.object({
  topic: z.string(),
  tone: z.string(),
  length: z.number(),
})
```

The resulting arguments are available inside the callback:

```typescript
async ({ topic, tone, length }) => {
  // Use the prompt arguments
}
```

---

# 7. Example: Simple Prompt

```typescript
server.prompt(
  {
    name: "explain",
    title: "Explain a Topic",
    description: "Generate a prompt that asks for a simple explanation",
    schema: z.object({
      topic: z.string(),
    }),
  },
  async ({ topic }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Explain ${topic} in simple terms.`,
          },
        },
      ],
    };
  }
);
```

A client can request the prompt with:

```typescript
const result = await session.getPrompt("explain", {
  topic: "MCP servers",
});
```

---

# 8. Standard `GetPromptResult`

The most direct way to return a prompt is to return a standard MCP `GetPromptResult`.

```typescript
return {
  messages: [
    {
      role: "user",
      content: {
        type: "text",
        text: "Explain this topic clearly.",
      },
    },
  ],
};
```

The `messages` array contains the messages that make up the generated prompt.

---

# 9. Prompt Messages

A prompt result uses MCP `PromptMessage` objects.

For example:

```typescript
return {
  messages: [
    {
      role: "user",
      content: {
        type: "text",
        text: "Explain artificial intelligence.",
      },
    },
  ],
};
```

Multiple messages can be returned:

```typescript
return {
  messages: [
    {
      role: "user",
      content: {
        type: "text",
        text: "You are an expert teacher.",
      },
    },
    {
      role: "user",
      content: {
        type: "text",
        text: "Explain machine learning to a beginner.",
      },
    },
  ],
};
```

---

# 10. Using `text()`

mcpfy also allows prompt callbacks to return the same content helpers used by tools.

For example:

```typescript
import { text } from "mcpfy-sdk/server";
```

A prompt can return:

```typescript
async ({ topic }) => {
  return text(`Explain ${topic} in simple terms.`);
}
```

mcpfy converts the returned content into the standard MCP prompt format.

---

# 11. Using `markdown()`

Markdown content can also be returned:

```typescript
import { markdown } from "mcpfy-sdk/server";
```

Example:

```typescript
async ({ topic }) => {
  return markdown(`
# Explain ${topic}

Provide:
- A simple definition
- Key concepts
- A practical example
  `);
}
```

mcpfy converts this content into a prompt message.

---

# 12. Using `object()`

Structured content can also be produced using the `object()` helper when appropriate.

```typescript
import { object } from "mcpfy-sdk/server";
```

Example:

```typescript
async ({ topic }) => {
  return object({
    instruction: "Explain the topic",
    topic,
    audience: "beginner",
  });
}
```

mcpfy converts content-helper results into prompt messages.

---

# 13. Callback Defined Inside the Definition

Instead of passing the callback as the second argument, it can be specified using `cb`.

```typescript
server.prompt({
  name: "code-review",
  description: "Generate a code review prompt",
  schema: z.object({
    language: z.string(),
  }),

  cb: async ({ language }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Review the following ${language} code for correctness and quality.`,
          },
        },
      ],
    };
  },
});
```

Both callback styles are supported.

---

# 14. Callback as the Second Argument

The callback can instead be separated from the definition:

```typescript
server.prompt(
  {
    name: "code-review",
    description: "Generate a code review prompt",
    schema: z.object({
      language: z.string(),
    }),
  },
  async ({ language }) => {
    return {
      messages: [
        {
          role: "user",
          content: {
            type: "text",
            text: `Review this ${language} code.`,
          },
        },
      ],
    };
  }
);
```

This style is useful when keeping configuration and implementation separate.

---

# 15. Using `ToolContext`

The prompt callback receives a `ToolContext` as its second parameter.

```typescript
server.prompt(
  {
    name: "context-aware",
  },
  async (params, ctx) => {
    // Use ctx when required
    return text("Generated prompt");
  }
);
```

The context is built using the same server context mechanism used by tool callbacks.

This allows prompt implementations to access the contextual capabilities provided by mcpfy.

For details about available context functionality, see the server/tool documentation.

---

# 16. Prompt Without Arguments

A prompt does not require a schema.

```typescript
server.prompt(
  {
    name: "daily-planning",
    description: "Generate a daily planning prompt",
  },
  async () => {
    return text(
      "Help the user create a practical plan for the day."
    );
  }
);
```

Because no schema is provided, the prompt is registered with an empty argument schema.

---

# 17. Prompt With Multiple Arguments

```typescript
server.prompt(
  {
    name: "write-email",
    description: "Generate an email-writing prompt",
    schema: z.object({
      recipient: z.string(),
      purpose: z.string(),
      tone: z.string(),
    }),
  },
  async ({ recipient, purpose, tone }) => {
    return text(
      `Write a ${tone} email to ${recipient} about ${purpose}.`
    );
  }
);
```

The MCP client can then request:

```typescript
const result = await session.getPrompt("write-email", {
  recipient: "project manager",
  purpose: "project status update",
  tone: "professional",
});
```

---

# 18. How Prompt Registration Works

Internally, `server.prompt()` delegates registration to the prompt registration layer.

The process is:

```text
server.prompt()
      │
      ▼
Validate callback
      │
      ▼
Convert Zod schema
      │
      ▼
Register with official MCP server
      │
      ▼
MCP client requests prompt
      │
      ▼
Execute callback
      │
      ▼
Convert result to GetPromptResult
      │
      ▼
Return prompt to client
```

mcpfy therefore provides a simpler developer-facing API while registering the prompt with the official MCP server implementation.

---

# 19. Automatic Result Conversion

Prompt callbacks can return either:

1. A standard `GetPromptResult`
2. A typed tool-style result
3. A `ToolContentResult`

If the callback already returns a `GetPromptResult`, mcpfy uses it directly.

For content-helper results, mcpfy converts the returned content into prompt messages.

Conceptually:

```text
Content result
     │
     ▼
content[]
     │
     ▼
PromptMessage[]
     │
     ▼
GetPromptResult
```

Each returned content item becomes a `user` prompt message.

---

# 20. Missing Callback Error

Every registered prompt must have a callback.

This is valid:

```typescript
server.prompt(
  {
    name: "example",
  },
  async () => {
    return text("Hello");
  }
);
```

This is also valid:

```typescript
server.prompt({
  name: "example",
  cb: async () => text("Hello"),
});
```

But the following is invalid:

```typescript
server.prompt({
  name: "example",
});
```

mcpfy throws an error indicating that the prompt is missing a callback.

---

# 21. Discovering Prompts from a Client

Once registered, prompts can be discovered using the mcpfy client:

```typescript
const prompts = await session.listPrompts();
```

Example:

```typescript
const prompts = await session.listPrompts();

for (const prompt of prompts) {
  console.log(prompt.name);
}
```

The client receives the prompt metadata exposed by the MCP server.

---

# 22. Requesting a Prompt from a Client

After discovering a prompt:

```typescript
const result = await session.getPrompt("summarize", {
  topic: "Model Context Protocol",
});
```

The server executes the registered callback and returns the generated prompt.

---

# 23. Complete Example

### Server

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "prompt-example",
  version: "1.0.0",
});

server.prompt(
  {
    name: "explain-topic",
    title: "Explain Topic",
    description: "Generate a beginner-friendly explanation prompt",
    schema: z.object({
      topic: z.string(),
    }),
  },
  async ({ topic }) => {
    return text(
      `Explain ${topic} in simple terms. Include a practical example.`
    );
  }
);

await server.listen();
```

### Client

```typescript
import { MCPClient } from "mcpfy-sdk/client";

const client = new MCPClient({
  mcpServers: {
    prompts: {
      command: "node",
      args: ["prompt-server.js"],
    },
  },
});

try {
  const session = await client.createSession("prompts");

  const prompts = await session.listPrompts();

  console.log("Available prompts:", prompts);

  const result = await session.getPrompt("explain-topic", {
    topic: "Model Context Protocol",
  });

  console.log("Prompt result:", result);
} finally {
  await client.closeAllSessions();
}
```

---

# 24. Best Practices

### Use descriptive prompt names

Prefer:

```typescript
name: "code-review"
```

over:

```typescript
name: "prompt1"
```

### Provide descriptions

Descriptions help MCP clients understand when a prompt should be used.

```typescript
description: "Generate a structured code review prompt"
```

### Use schemas for arguments

If a prompt requires input, define it explicitly:

```typescript
schema: z.object({
  language: z.string(),
  code: z.string(),
})
```

### Keep callbacks focused

A prompt callback should primarily construct the prompt rather than contain unnecessary application logic.

### Use content helpers when appropriate

For simple prompts, helpers such as:

```typescript
text(...)
markdown(...)
object(...)
```

can make the implementation shorter and easier to read.

---

# 25. API Summary

## `server.prompt()`

Registers an MCP prompt.

```typescript
server.prompt(
  definition,
  callback?
);
```

## `PromptDefinition`

```typescript
interface PromptDefinition<TInput = Record<string, any>> {
  name: string;
  title?: string;
  description?: string;
  schema?: z.ZodObject<any>;
  cb?: PromptCallback<TInput>;
}
```

## `PromptCallback`

```typescript
type PromptCallback<TInput = Record<string, any>> = (
  params: TInput,
  ctx: ToolContext
) => Promise<
  GetPromptResult |
  TypedCallToolResult<any> |
  ToolContentResult
>;
```

## Client prompt methods

```typescript
await session.listPrompts();

await session.getPrompt(name, args);
```

---

# 26. Summary

mcpfy makes MCP prompt development straightforward by providing a declarative `.prompt()` API with:

* Prompt metadata
* Zod-based argument schemas
* Typed callback parameters
* `ToolContext` access
* Standard MCP `GetPromptResult` support
* `text()`, `markdown()`, and `object()` content-helper support
* Automatic conversion of content results into MCP prompt messages

The typical implementation is:

```typescript
server.prompt(
  {
    name: "my-prompt",
    description: "Generate a useful prompt",
    schema: z.object({
      topic: z.string(),
    }),
  },
  async ({ topic }) => {
    return text(`Explain ${topic} clearly.`);
  }
);
```

The resulting prompt can then be discovered and requested by any compatible MCP client.