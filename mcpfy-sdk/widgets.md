# Widgets

Widgets allow an MCP server to expose interactive user interfaces alongside MCP tools. A widget can be used when a tool needs to present richer output than plain text or structured data.

mcpfy provides the widget runtime, registration APIs, and build tooling required to develop and serve widgets.

## Widget Directory Structure

By default, mcpfy looks for widgets under:

```text
src/widgets
```

Each widget has its own directory.

A typical project looks like:

```text
my-mcp-server/
├── package.json
├── tsconfig.json
└── src/
    ├── server.ts
    └── widgets/
        └── weather/
            ├── main.tsx
            └── ...
```

The widget name is determined by its directory name. In this example, the widget is named `weather`.

The widget directory can be changed through the server's `widgetsDir` configuration:

```typescript
const server = new MCPServer({
  name: "weather-server",
  version: "1.0.0",
  widgetsDir: "src/widgets",
});
```

If `widgetsDir` is not specified, the default is `src/widgets`.

## Registering a Widget

A widget is associated with a tool using the `widget` property.

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
    description: "Get the current weather",
    schema: z.object({
      city: z.string(),
    }),
    widget: "weather",
  },
  async ({ city }) => {
    return object({
      city,
      temperature: 24,
      condition: "Sunny",
    });
  }
);

await server.listen();
```

The value:

```typescript
widget: "weather"
```

refers to the widget directory:

```text
src/widgets/weather/
```

**The widget entry file must exist before the server starts.** Registering a widget name without the corresponding widget directory and entry point can cause the server to fail during startup.

## Creating the Widget Entry File

Create:

```text
src/widgets/weather/main.tsx
```

The entry file contains the widget UI.

A basic React widget can be structured around the SDK's widget runtime:

```tsx
import React from "react";

export default function Weather() {
  return (
    <div>
      <h1>Weather</h1>
      <p>Weather information will appear here.</p>
    </div>
  );
}
```

The standard `src/widgets/<name>/main.tsx` convention is handled by the mcpfy widget runtime.

You should **not manually add another `ThemeProvider` or `HostRuntime` around the standard widget entry point** when using the normal SDK convention. The runtime handles the required provider setup.

## Widget Content

Widgets can provide HTML content or reference a URL.

When using HTML content, the content must use the structured form:

```typescript
{
  type: "html",
  html: "<div>Hello</div>",
}
```

For URL-based content:

```typescript
{
  type: "url",
  url: "https://example.com/widget",
}
```

Do not provide the HTML as a bare string:

```typescript
content: "<html>...</html>"
```

The content type must explicitly identify whether the widget content is HTML or a URL.

## Widget Size

Widget dimensions are represented as a tuple containing width and height.

For example:

```typescript
size: ["800px", "600px"]
```

The two values represent:

1. Width
2. Height

Do not use:

```typescript
size: "full"
```

because the SDK expects the size tuple.

## Building Widgets

Widget source code must be built before it can be used in a production server.

mcpfy provides CLI commands for this:

```bash
mcpfy dev
```

and:

```bash
mcpfy build
```

### Development

Use:

```bash
mcpfy dev
```

while developing widgets.

This starts the widget development workflow and rebuilds the widget as changes are made.

### Production Build

Before starting the server in production, run:

```bash
mcpfy build
```

The production server expects the widget assets to have been built.

Therefore, a production deployment should include the widget build step before starting the server.

## Widget Development Workflow

A typical workflow is:

### 1. Create the widget directory

```text
src/widgets/weather/
```

### 2. Create the entry file

```text
src/widgets/weather/main.tsx
```

### 3. Register the widget with a tool

```typescript
widget: "weather"
```

### 4. Run the widget development command

```bash
mcpfy dev
```

### 5. Build for production

```bash
mcpfy build
```

### 6. Start the MCP server

Start the server using the project's normal start command after the widget build has completed.

## Example Project

A complete project can look like:

```text
my-mcp-server/
├── package.json
├── tsconfig.json
└── src/
    ├── server.ts
    └── widgets/
        └── weather/
            ├── main.tsx
            └── ...
```

The server registers the widget:

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
      temperature: 24,
      condition: "Sunny",
    });
  }
);

await server.listen({
  transport: "http",
  port: 4000,
});
```

The corresponding widget entry point is:

```tsx
import React from "react";

export default function Weather() {
  return (
    <div>
      <h1>Weather</h1>
      <p>Weather information will appear here.</p>
    </div>
  );
}
```

## Widget Runtime

The widget runtime provides the environment required for widgets to communicate with the MCP host.

For the standard widget structure, mcpfy takes care of the required runtime/provider setup.

This means a widget entry point should focus on the UI rather than manually recreating the host runtime.

For advanced React integration, see [Widget React](widget-react.md).

## Widget and Tool Communication

A widget is commonly paired with an MCP tool.

The general flow is:

```text
MCP Client
    │
    │ calls tool
    ▼
MCP Server
    │
    │ executes tool
    ▼
Tool Result
    │
    │ associated widget
    ▼
Widget UI
    │
    │ renders interactive content
    ▼
User
```

The tool performs the server-side operation, while the widget provides the user-facing interface.

## Production Considerations

When deploying an MCP server that uses widgets:

1. Ensure every registered widget has a corresponding widget directory.
2. Ensure each widget has its required `main.tsx` entry point.
3. Run `mcpfy build` before starting the production server.
4. Include the generated widget assets in the deployment.
5. Keep widget names consistent between `server.tool()` and the widget directory.
6. Do not manually double-wrap the standard widget entry point with providers already supplied by the SDK runtime.

## Troubleshooting

### Widget Not Found

If the server cannot find a registered widget, verify that:

```typescript
widget: "weather"
```

matches:

```text
src/widgets/weather/
```

Also verify that:

```text
src/widgets/weather/main.tsx
```

exists.

### Widget Fails During Production Startup

Make sure the widget has been built:

```bash
mcpfy build
```

The production server requires the generated widget assets.

### Invalid Widget Content

Use the structured content forms:

```typescript
{
  type: "html",
  html: "<div>...</div>",
}
```

or:

```typescript
{
  type: "url",
  url: "https://example.com",
}
```

Do not pass raw HTML as a string.

### Invalid Widget Size

Use a width/height tuple:

```typescript
size: ["800px", "600px"]
```

rather than:

```typescript
size: "full"
```

## Summary

Widgets provide an interactive UI layer for MCP applications.

The core workflow is:

```text
Create widget directory
        ↓
Create main.tsx
        ↓
Register widget with a tool
        ↓
Develop with mcpfy dev
        ↓
Build with mcpfy build
        ↓
Start the MCP server
```

The standard widget convention is:

```text
src/widgets/<widget-name>/main.tsx
```

and the tool references it using:

```typescript
widget: "<widget-name>"
```

This convention allows mcpfy to locate, build, and serve the widget as part of the MCP application.
