# MCP Resources

mcpfy provides a simple API for exposing **MCP resources** from a server.

Resources allow an MCP server to make data available to MCP clients through URIs. mcpfy supports both:

* **Static resources** — resources with a fixed URI.
* **Resource templates** — dynamic resources whose URI contains variables.

Resource handlers can return standard MCP resource results or use mcpfy's content helpers such as `text()`, `markdown()`, and `object()`.

---

# 1. Overview

A resource represents data that can be read by an MCP client.

A static resource has a fixed URI:

```text
config://app
```

A resource template can generate resources dynamically:

```text
user://{userId}/profile
```

The basic static-resource pattern is:

```typescript
server.resource(
  {
    name: "app-config",
    uri: "config://app",
  },
  async (ctx) => {
    return text("Application configuration");
  }
);
```

---

# 2. Importing Resource APIs

Resource functionality is available from:

```typescript
import {
  MCPServer,
  text,
  markdown,
  object,
  type ResourceDefinition,
  type ReadResourceCallback,
  type FlatResourceTemplateDefinition,
  type ReadResourceTemplateCallback,
} from "mcpfy-sdk/server";
```

---

# 3. Static Resources

A static resource has a predefined URI.

Example:

```typescript
server.resource(
  {
    name: "app-info",
    uri: "info://application",
    title: "Application Information",
    description: "Information about the application",
    mimeType: "text/plain",
  },
  async () => {
    return text("This application provides MCP services.");
  }
);
```

The resource can then be read by an MCP client using its URI.

---

# 4. Resource Definition

The `ResourceDefinition` interface is:

```typescript
interface ResourceDefinition {
  name: string;
  uri: string;
  title?: string;
  description?: string;
  mimeType?: string;
  readCallback?: ReadResourceCallback;
}
```

### Properties

| Property       | Type                   | Required | Description                            |
| -------------- | ---------------------- | -------: | -------------------------------------- |
| `name`         | `string`               |      Yes | Resource name                          |
| `uri`          | `string`               |      Yes | Fixed resource URI                     |
| `title`        | `string`               |       No | Human-readable title                   |
| `description`  | `string`               |       No | Description of the resource            |
| `mimeType`     | `string`               |       No | MIME type of the resource              |
| `readCallback` | `ReadResourceCallback` |       No | Function used to generate the resource |

A callback must be supplied either through `readCallback` or as the second argument to `.resource()`.

---

# 5. Registering a Resource

The standard API is:

```typescript
server.resource(definition, callback?);
```

Example:

```typescript
server.resource(
  {
    name: "server-status",
    uri: "status://server",
    description: "Current server status",
  },
  async () => {
    return text("Server is running.");
  }
);
```

---

# 6. Defining the Callback Inside the Definition

The callback can also be provided through `readCallback`:

```typescript
server.resource({
  name: "server-status",
  uri: "status://server",
  description: "Current server status",
  readCallback: async () => {
    return text("Server is running.");
  },
});
```

Both styles are supported.

---

# 7. Resource Callback

A static resource callback receives the mcpfy `ToolContext`:

```typescript
type ReadResourceCallback = (
  ctx: ToolContext
) => Promise<ResourceResult>;
```

Example:

```typescript
server.resource(
  {
    name: "user-info",
    uri: "user://current",
  },
  async (ctx) => {
    return text("Current user information");
  }
);
```

The context allows the resource implementation to use the contextual capabilities provided by mcpfy.

---

# 8. Returning Standard MCP Resource Results

A resource callback can return a standard MCP `ReadResourceResult`.

For example:

```typescript
return {
  contents: [
    {
      uri: "info://application",
      mimeType: "text/plain",
      text: "Application information",
    },
  ],
};
```

When a standard `ReadResourceResult` is returned, mcpfy passes it through directly.

---

# 9. Using `text()`

For simple text resources, use the `text()` helper:

```typescript
import { text } from "mcpfy-sdk/server";
```

Example:

```typescript
server.resource(
  {
    name: "documentation",
    uri: "docs://intro",
  },
  async () => {
    return text("Welcome to the application documentation.");
  }
);
```

mcpfy converts the returned content into the appropriate MCP resource format.

---

# 10. Using `markdown()`

Markdown resources can be created using:

```typescript
import { markdown } from "mcpfy-sdk/server";
```

Example:

```typescript
server.resource(
  {
    name: "guide",
    uri: "docs://guide",
    mimeType: "text/markdown",
  },
  async () => {
    return markdown(`
# Application Guide

## Getting Started

Follow these steps to get started.

- Install the application
- Configure the server
- Start the MCP server
    `);
  }
);
```

---

# 11. Using `object()`

Structured data can be returned using the `object()` helper:

```typescript
import { object } from "mcpfy-sdk/server";
```

Example:

```typescript
server.resource(
  {
    name: "server-config",
    uri: "config://server",
    mimeType: "application/json",
  },
  async () => {
    return object({
      host: "localhost",
      port: 3000,
      transport: "http",
    });
  }
);
```

mcpfy converts content-helper results into an MCP-compatible resource response.

---

# 12. MIME Types

A resource can specify its MIME type:

```typescript
mimeType: "text/plain"
```

For example:

```typescript
server.resource(
  {
    name: "data",
    uri: "data://example",
    mimeType: "application/json",
  },
  async () => {
    return object({
      message: "Hello",
    });
  }
);
```

If a MIME type is supplied in the resource definition, mcpfy uses it when converting content-helper results.

For text content without an explicit MIME type, mcpfy falls back to:

```text
text/plain
```

---

# 13. Resource Templates

Resource templates are used when the resource URI contains dynamic variables.

For example:

```text
user://{userId}/profile
```

Instead of registering every user individually, a single resource template can handle all users.

---

# 14. Resource Template Definition

The `FlatResourceTemplateDefinition` interface is:

```typescript
interface FlatResourceTemplateDefinition<
  TParams extends Record<string, any> = Record<string, any>
> {
  name: string;
  uriTemplate: string;
  title?: string;
  description?: string;
  mimeType?: string;
  schema?: z.ZodTypeAny;
  readCallback?: ReadResourceTemplateCallback<TParams>;
}
```

### Properties

| Property       | Type                           | Required | Description                       |
| -------------- | ------------------------------ | -------: | --------------------------------- |
| `name`         | `string`                       |      Yes | Template name                     |
| `uriTemplate`  | `string`                       |      Yes | URI template containing variables |
| `title`        | `string`                       |       No | Human-readable title              |
| `description`  | `string`                       |       No | Template description              |
| `mimeType`     | `string`                       |       No | MIME type                         |
| `schema`       | `z.ZodTypeAny`                 |       No | Type hint for template variables  |
| `readCallback` | `ReadResourceTemplateCallback` |       No | Template read callback            |

---

# 15. Registering a Resource Template

Use:

```typescript
server.resourceTemplate(definition, callback?);
```

Example:

```typescript
server.resourceTemplate(
  {
    name: "user-profile",
    uriTemplate: "user://{userId}/profile",
    title: "User Profile",
    description: "Retrieve a user's profile",
    mimeType: "application/json",
  },
  async (uri, params) => {
    return object({
      userId: params.userId,
      name: "Example User",
    });
  }
);
```

---

# 16. Template Callback

A resource-template callback receives three arguments:

```typescript
(uri, params, ctx)
```

Its type is:

```typescript
type ReadResourceTemplateCallback<
  TParams extends Record<string, any> = Record<string, any>
> = (
  uri: URL,
  params: TParams,
  ctx: ToolContext
) => Promise<ResourceResult>;
```

### Arguments

#### `uri`

The complete requested resource URI.

```typescript
uri: URL
```

#### `params`

The variables extracted from the URI template.

```typescript
params: TParams
```

#### `ctx`

The mcpfy `ToolContext`.

```typescript
ctx: ToolContext
```

---

# 17. Template Example

Consider:

```typescript
uriTemplate: "user://{userId}/profile"
```

A client requesting:

```text
user://123/profile
```

causes the callback to receive variables corresponding to:

```typescript
{
  userId: "123"
}
```

Example:

```typescript
server.resourceTemplate(
  {
    name: "user-profile",
    uriTemplate: "user://{userId}/profile",
  },
  async (uri, params) => {
    return object({
      requestedUri: uri.toString(),
      userId: params.userId,
    });
  }
);
```

---

# 18. Template Variables and `schema`

A resource template can optionally specify a Zod schema:

```typescript
import { z } from "zod";

server.resourceTemplate(
  {
    name: "user-profile",
    uriTemplate: "user://{userId}/profile",
    schema: z.object({
      userId: z.string(),
    }),
  },
  async (uri, params) => {
    return object({
      userId: params.userId,
    });
  }
);
```

In the current implementation, the `schema` is a **type hint only**. It is not used for runtime validation of template variables.

The URI template itself is matched using the official MCP SDK's `ResourceTemplate` implementation.

---

# 19. Dynamic Data Example

Resource templates are useful when resource content depends on the requested URI.

```typescript
server.resourceTemplate(
  {
    name: "product",
    uriTemplate: "product://{productId}",
    mimeType: "application/json",
  },
  async (uri, params) => {
    const productId = params.productId;

    return object({
      id: productId,
      name: `Product ${productId}`,
      uri: uri.toString(),
    });
  }
);
```

A request for:

```text
product://42
```

can produce:

```json
{
  "id": "42",
  "name": "Product 42",
  "uri": "product://42"
}
```

---

# 20. Returning Text From a Template

```typescript
server.resourceTemplate(
  {
    name: "user-summary",
    uriTemplate: "user://{userId}/summary",
  },
  async (_uri, params) => {
    return text(`Summary for user ${params.userId}`);
  }
);
```

---

# 21. Returning Markdown From a Template

```typescript
server.resourceTemplate(
  {
    name: "user-report",
    uriTemplate: "user://{userId}/report",
    mimeType: "text/markdown",
  },
  async (_uri, params) => {
    return markdown(`
# User Report

User ID: ${params.userId}

This report contains information about the requested user.
    `);
  }
);
```

---

# 22. Returning Images or Audio

mcpfy's resource-result conversion also supports content items representing images and audio.

For image or audio content, the resulting MCP resource contains binary data through the resource `blob` field and uses the item's MIME type.

This allows resource handlers to expose non-text content when the corresponding content result is available.

---

# 23. Automatic Result Conversion

Resource callbacks can return either a standard MCP `ReadResourceResult` or mcpfy content-helper results.

Conceptually:

```text
Resource callback
       │
       ▼
 ResourceResult
       │
       ├── ReadResourceResult
       │       └── returned directly
       │
       └── Content result
               │
               ▼
       Convert to MCP contents[]
               │
               ▼
       ReadResourceResult
```

For text content:

```typescript
text("Hello")
```

is converted into a resource content entry containing:

```typescript
{
  uri,
  mimeType,
  text: "Hello"
}
```

For image and audio content, the data is represented as a resource `blob`.

---

# 24. Resource URI

A resource must have a URI.

Example:

```typescript
uri: "config://application"
```

Resource templates instead define a URI pattern:

```typescript
uriTemplate: "user://{userId}/profile"
```

Use a static resource when the URI is fixed and a resource template when part of the URI needs to be resolved dynamically.

---

# 25. Static Resource vs Resource Template

| Feature            | Static Resource     | Resource Template           |
| ------------------ | ------------------- | --------------------------- |
| API                | `server.resource()` | `server.resourceTemplate()` |
| URI                | Fixed               | Dynamic                     |
| Variables          | No                  | Yes                         |
| Callback arguments | `ctx`               | `uri`, `params`, `ctx`      |
| Zod schema         | Not applicable      | Optional type hint          |
| Best for           | Fixed data          | User/item-specific data     |

---

# 26. Resource Subscriptions

MCP resources can support subscriptions when clients need to be notified that a resource has changed.

Subscriptions are useful for resources whose contents can change while the server is running.

A client can subscribe to a resource using the MCP resource-subscription mechanism. When the resource changes, the server can notify subscribed clients by refreshing the resource.

mcpfy exposes resource refresh methods on `MCPServer` for this purpose.

### Refreshing a Specific Resource

Use:

```typescript
await server.refreshResource(uri);
```

For example:

```typescript
server.resource(
  {
    name: "server-status",
    uri: "status://server",
    mimeType: "text/plain",
  },
  async () => {
    return text(getCurrentStatus());
  }
);

// After the underlying data changes
await server.refreshResource("status://server");
```

The refresh operation tells the underlying MCP server that the resource has changed so subscribed clients can request the latest contents.

### Refreshing Multiple Resources

When multiple resources need to be refreshed, use:

```typescript
await server.refreshResources(uris);
```

For example:

```typescript
await server.refreshResources([
  "status://server",
  "config://server",
]);
```

Use `refreshResource()` when one resource changes and `refreshResources()` when several resources need to be invalidated together.

> Resource subscriptions are useful only when the MCP client supports the corresponding subscription capability.

---

# 27. Missing Callback

A resource must have a read callback.

This is invalid:

```typescript
server.resource({
  name: "example",
  uri: "example://data",
});
```

mcpfy throws an error indicating that the resource has no read callback.

The same rule applies to resource templates:

```typescript
server.resourceTemplate({
  name: "example",
  uriTemplate: "example://{id}",
});
```

A callback must be supplied either through `readCallback` or the second argument.

---

# 28. Complete Static Resource Example

```typescript
import { MCPServer, text } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "resource-server",
  version: "1.0.0",
});

server.resource(
  {
    name: "application-info",
    uri: "info://application",
    title: "Application Information",
    description: "Basic information about the application",
    mimeType: "text/plain",
  },
  async () => {
    return text(
      "This application provides MCP resources using mcpfy."
    );
  }
);

await server.listen();
```

---

# 29. Complete Resource Template Example

```typescript
import { MCPServer, object } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "resource-template-server",
  version: "1.0.0",
});

server.resourceTemplate(
  {
    name: "user-profile",
    uriTemplate: "user://{userId}/profile",
    title: "User Profile",
    description: "Retrieve a user profile",
    mimeType: "application/json",
    schema: z.object({
      userId: z.string(),
    }),
  },
  async (uri, params) => {
    return object({
      userId: params.userId,
      requestedUri: uri.toString(),
    });
  }
);

await server.listen();
```

---

# 30. How Resources Work Internally

When a static resource is registered:

```text
server.resource()
       │
       ▼
registerResource()
       │
       ▼
Official MCP server
       │
       ▼
Client requests URI
       │
       ▼
Read callback executes
       │
       ▼
Result converted if necessary
       │
       ▼
ReadResourceResult returned
```

For a resource template:

```text
server.resourceTemplate()
       │
       ▼
ResourceTemplate created
       │
       ▼
Official MCP server
       │
       ▼
Client requests dynamic URI
       │
       ▼
Template variables resolved
       │
       ▼
Callback(uri, params, ctx)
       │
       ▼
Result converted if necessary
       │
       ▼
ReadResourceResult returned
```

For subscribed resources:

```text
Client subscribes to resource
       │
       ▼
Resource changes
       │
       ▼
server.refreshResource()
       │
       ▼
MCP resource-updated notification
       │
       ▼
Client requests latest resource
```

---

# 31. Best Practices

### Use meaningful resource names

Prefer:

```typescript
name: "user-profile"
```

over:

```typescript
name: "resource1"
```

### Use descriptive URIs

Prefer:

```text
user://{userId}/profile
```

over:

```text
data://{id}
```

when the resource represents user profiles.

### Provide descriptions

Descriptions help clients understand what a resource represents.

```typescript
description: "Retrieve the profile information for a user"
```

### Set an appropriate MIME type

For example:

```typescript
mimeType: "application/json"
```

for JSON-like data and:

```typescript
mimeType: "text/markdown"
```

for Markdown content.

### Use templates for dynamic resources

If many resources follow the same URI pattern, use `resourceTemplate()` instead of registering each URI individually.

### Refresh changing resources

If a resource changes while the server is running and clients may subscribe to it, call:

```typescript
await server.refreshResource(uri);
```

or:

```typescript
await server.refreshResources(uris);
```

after the underlying data changes.

---

# 32. API Summary

### `server.resource()`

Registers a static MCP resource.

```typescript
server.resource(
  definition,
  callback?
);
```

### `server.resourceTemplate()`

Registers a dynamic MCP resource template.

```typescript
server.resourceTemplate(
  definition,
  callback?
);
```

### `server.refreshResource()`

Notifies the underlying MCP server that a specific resource has changed.

```typescript
await server.refreshResource(uri);
```

### `server.refreshResources()`

Refreshes multiple resources.

```typescript
await server.refreshResources(uris);
```

### `ResourceDefinition`

```typescript
interface ResourceDefinition {
  name: string;
  uri: string;
  title?: string;
  description?: string;
  mimeType?: string;
  readCallback?: ReadResourceCallback;
}
```

### `ReadResourceCallback`

```typescript
type ReadResourceCallback = (
  ctx: ToolContext
) => Promise<ResourceResult>;
```

### `FlatResourceTemplateDefinition`

```typescript
interface FlatResourceTemplateDefinition<
  TParams extends Record<string, any> = Record<string, any>
> {
  name: string;
  uriTemplate: string;
  title?: string;
  description?: string;
  mimeType?: string;
  schema?: z.ZodTypeAny;
  readCallback?: ReadResourceTemplateCallback<TParams>;
}
```

### `ReadResourceTemplateCallback`

```typescript
type ReadResourceTemplateCallback<
  TParams extends Record<string, any> = Record<string, any>
> = (
  uri: URL,
  params: TParams,
  ctx: ToolContext
) => Promise<ResourceResult>;
```

---

# 33. Summary

mcpfy simplifies MCP resource development by providing:

* Static resource registration with `server.resource()`
* Dynamic resource templates with `server.resourceTemplate()`
* Optional resource metadata
* MIME type support
* Zod type hints for resource-template variables
* `ToolContext` access
* Standard MCP `ReadResourceResult` support
* `text()`, `markdown()`, and `object()` content helpers
* Automatic conversion of content results into MCP resource contents
* Resource subscriptions through the MCP resource-subscription mechanism
* `server.refreshResource()` for refreshing an individual resource
* `server.refreshResources()` for refreshing multiple resources

For fixed data, use:

```typescript
server.resource(...)
```

For dynamic URI-based data, use:

```typescript
server.resourceTemplate(...)
```

For a resource whose contents can change while the server is running, use the resource subscription mechanism together with:

```typescript
await server.refreshResource(uri);
```

or:

```typescript
await server.refreshResources(uris);
```

Both resource APIs ultimately integrate with the official MCP server implementation while providing a cleaner developer experience.
