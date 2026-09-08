
# Widgets

MCPfy provides a unified way to build interactive UI widgets that can be attached to MCP tools and rendered by compatible MCP hosts.

Widgets can work across multiple UI protocols, including:

* MCP-UI
* MCP Apps
* OpenAI Apps SDK

The SDK handles protocol-specific metadata and content generation so that the same widget can be exposed through a single MCPfy tool definition.

---

## Overview

A widget is an interactive UI resource associated with an MCP tool.

Instead of returning only text from a tool, a widget-enabled tool can return:

1. The tool's structured result.
2. Text content representing the result.
3. UI content for compatible hosts.

A widget can therefore provide a richer experience while retaining normal MCP tool behavior.

The recommended approach is to define widgets through the `widget` option on `server.tool()`.

The older `.widget()` server method is still available for compatibility but is deprecated.

---

## Widget Directory

MCPfy supports file-based widgets stored in a widgets directory.

By default, the SDK looks for widgets under:

```text
src/widgets
```

You can change the directory through `widgetsDir` when creating the server:

```ts
import { MCPServer } from "mcpfy-sdk/server";

const server = new MCPServer({
  name: "My Server",
  version: "1.0.0",
  widgetsDir: "./src/widgets",
});
```

The directory is used when resolving widget references such as:

```ts
widget: "weather"
```

The SDK prepares registered widgets before the server starts listening.

---

## Creating a Widget

A widget is normally associated with a tool.

For example:

```ts
import { MCPServer } from "mcpfy-sdk/server";
import { z } from "zod";

const server = new MCPServer({
  name: "Weather Server",
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
    return {
      city,
      temperatureC: 24,
      condition: "Sunny",
    };
  }
);

await server.listen();
```

Here:

* `weather` is the MCP tool.
* `widget: "weather"` associates the tool with the widget named `weather`.
* The callback produces structured data.
* MCPfy exposes the appropriate UI metadata for supported protocols.

---

## Widget Protocols

MCPfy can expose widgets through multiple protocols.

The supported protocol names are:

```ts
type WidgetProtocol = "mcp-ui" | "mcp-apps" | "apps-sdk";
```

When protocols are not explicitly specified, MCPfy uses all supported protocols by default.

You can restrict a widget to specific protocols:

```ts
widget: {
  dir: "weather",
  protocols: ["mcp-ui", "mcp-apps"],
}
```

This is useful when a widget depends on functionality that is available only in particular hosts.

---

## Widget Configuration

A widget can be configured using either a directory name or a widget options object.

The directory form is the simplest:

```ts
widget: "weather"
```

For more control:

```ts
widget: {
  dir: "weather",
  protocols: ["mcp-ui", "mcp-apps", "apps-sdk"],
  csp: {
    connectDomains: ["https://api.example.com"],
  },
}
```

The exact options available depend on the widget definition types exposed by MCPfy.

---

## Widget Content

MCPfy can generate the UI content required by the supported widget protocols.

For MCP-UI, the server creates an appropriate UI content block when the widget is invoked.

The tool response still contains the structured data:

```ts
return {
  city,
  temperatureC: 24,
  condition: "Sunny",
};
```

This means the widget does not replace the underlying MCP tool result. It adds a UI representation on top of it.

---

## Content Security Policy

Widgets can define Content Security Policy settings for external resources.

For example:

```ts
widget: {
  dir: "weather",
  csp: {
    connectDomains: ["https://api.example.com"],
  },
}
```

Use CSP configuration when the widget needs to communicate with external services.

Keep the allowed domains as narrow as possible.

---

## Widget Size

Widget definitions can optionally specify a preferred size.

For example:

```ts
widget: {
  dir: "dashboard",
  size: "full",
}
```

The available size values depend on the widget types supported by the installed MCPfy version.

When no size is specified, the host can use its default rendering behavior.

---

## Legacy `.widget()` API

MCPfy also exposes a `.widget()` method on `MCPServer`:

```ts
server.widget(
  {
    name: "weather",
    description: "Weather widget",
    content: "...",
  },
  async (params, ctx) => {
    return {
      temperatureC: 24,
    };
  }
);
```

This API is **deprecated**.

The recommended approach is to associate a widget with `server.tool()`:

```ts
server.tool({
  name: "weather",
  widget: "weather",
  // ...
});
```

The legacy `.widget()` API remains available for one release for compatibility.

---

## Widget Callbacks

A widget callback receives the tool input and a `ToolContext`.

For example:

```ts
server.widget(
  {
    name: "weather",
    content: "<html>...</html>",
  },
  async (params, ctx) => {
    return {
      city: params.city,
      temperatureC: 24,
    };
  }
);
```

The returned value becomes the widget's structured tool result.

---

## Widget Response

When a widget-enabled tool is called, MCPfy produces a response containing structured data.

For example:

```ts
{
  city: "Delhi",
  temperatureC: 24,
  condition: "Sunny"
}
```

For MCP-UI-enabled widgets, the response also includes the generated UI content block.

This allows MCP clients that do not support interactive widgets to continue using the underlying tool result.

---

## React Widgets

MCPfy provides a React runtime for building widgets that communicate with their host.

Import React widget functionality from:

```ts
import {
  HostRuntime,
  ThemeProvider,
  useHostContext,
  useCallTool,
} from "mcpfy-sdk/widget";
```

The React runtime provides hooks for:

* Host protocol detection
* Tool calls
* Tool results
* Host context
* Theme
* Layout modes
* Widget state
* Model context
* Follow-up messages
* External links
* View tools

See the dedicated **Widget React** documentation for the complete React API.

---

## Host Protocol Detection

Widgets can determine which protocol they are running under:

```ts
import { useHostProtocol } from "mcpfy-sdk/widget";

function Widget() {
  const protocol = useHostProtocol();

  return <div>Protocol: {protocol}</div>;
}
```

The runtime can detect:

```text
apps-sdk
mcp-apps
mcp-ui
none
```

`none` indicates that no supported host protocol was detected.

---

## Calling MCP Tools from a Widget

Use `useCallTool()` to call another MCP tool from the widget:

```tsx
import { useCallTool } from "mcpfy-sdk/widget";

export function WeatherWidget() {
  const getWeather = useCallTool("weather");

  return (
    <button onClick={() => getWeather.call({ city: "Delhi" })}>
      Get weather
    </button>
  );
}
```

The returned handle provides:

```ts
{
  call,
  isPending,
  data,
  error
}
```

For example:

```tsx
const weather = useCallTool("weather");

if (weather.isPending) {
  return <div>Loading...</div>;
}

if (weather.error) {
  return <div>Error loading weather</div>;
}

return <div>{JSON.stringify(weather.data)}</div>;
```

---

## Linked Tool

A widget can access the tool that originally mounted it with `useLinkedTool()`:

```tsx
import { useLinkedTool } from "mcpfy-sdk/widget";

function Widget() {
  const tool = useLinkedTool();

  return (
    <button onClick={() => tool.call()}>
      Refresh
    </button>
  );
}
```

The hook returns:

```ts
{
  name: string;
  call: (args?) => Promise<unknown>;
}
```

---

## Host Context

Use `useHostContext()` to access information supplied by the host:

```tsx
import { useHostContext } from "mcpfy-sdk/widget";

function Widget() {
  const host = useHostContext();

  return (
    <div>
      <p>Protocol: {host.protocol}</p>
      <p>Layout: {host.layoutMode}</p>
      <p>Locale: {host.locale}</p>
    </div>
  );
}
```

The context includes:

```ts
{
  protocol,
  layoutMode,
  locale,
  platform,
  capabilities
}
```

---

## Theme

MCPfy provides theme support through `ThemeProvider`.

```tsx
import { ThemeProvider, HostRuntime } from "mcpfy-sdk/widget";

export function App() {
  return (
    <ThemeProvider>
      <HostRuntime toolName="weather">
        <WeatherWidget />
      </HostRuntime>
    </ThemeProvider>
  );
}
```

The current theme can be read using:

```tsx
import { useHostTheme } from "mcpfy-sdk/widget";

function Widget() {
  const theme = useHostTheme();

  return <div>Current theme: {theme}</div>;
}
```

Supported themes are:

```text
light
dark
```

The runtime initially falls back to the browser's preferred color scheme when host theme information is unavailable.

---

## Layout Modes

Widgets can inspect and request layout modes:

```tsx
import { useLayoutMode } from "mcpfy-sdk/widget";

function Widget() {
  const layout = useLayoutMode();

  return (
    <button onClick={() => layout.request("fullscreen")}>
      Expand
    </button>
  );
}
```

The hook provides:

```ts
{
  mode,
  request,
  available
}
```

`available` contains the layout modes supported by the current host.

The host ultimately determines whether a requested mode can be applied.

---

## Widget State

Widgets can persist state through the host when supported.

```tsx
import { useWidgetState } from "mcpfy-sdk/widget";

function Widget() {
  const { state, setState } = useWidgetState();

  return (
    <button
      onClick={() =>
        setState({
          selectedCity: "Delhi",
        })
      }
    >
      Select Delhi
    </button>
  );
}
```

The state is represented as:

```ts
Record<string, unknown> | undefined
```

The actual persistence behavior depends on the host protocol.

---

## View State

For local widget state that should also be synchronized with host state and model context, use `useViewState()`:

```tsx
import { useViewState } from "mcpfy-sdk/widget";

function Widget() {
  const [state, setState] = useViewState({
    selectedCity: "Delhi",
  });

  return (
    <button
      onClick={() =>
        setState((previous) => ({
          ...previous,
          selectedCity: "Mumbai",
        }))
      }
    >
      Change city
    </button>
  );
}
```

`useViewState()`:

1. Initializes local state.
2. Restores previously available widget state.
3. Updates local state.
4. Persists the state through the host adapter.
5. Publishes the state as model context when supported.

---

## Sending Follow-Up Messages

Widgets can request a follow-up message:

```tsx
import { useSendFollowUp } from "mcpfy-sdk/widget";

function Widget() {
  const sendFollowUp = useSendFollowUp();

  return (
    <button onClick={() => sendFollowUp("Tell me more about this result")}>
      Ask for more
    </button>
  );
}
```

MCPfy uses the host's native follow-up mechanism when available and falls back to the appropriate protocol-specific behavior.

---

## Opening External Links

Use `useOpenExternal()` instead of directly relying on browser APIs:

```tsx
import { useOpenExternal } from "mcpfy-sdk/widget";

function Widget() {
  const openExternal = useOpenExternal();

  return (
    <button
      onClick={() => openExternal("https://example.com")}
    >
      Open website
    </button>
  );
}
```

MCPfy delegates the operation to the host when the host provides an external-link API.

---

## Model Context

Widgets can publish information back to the model when the host supports it.

```tsx
import { useModelContext } from "mcpfy-sdk/widget";

function Widget() {
  const modelContext = useModelContext();

  async function publish() {
    await modelContext.publish({
      text: "The user selected Delhi.",
      structuredContent: {
        city: "Delhi",
      },
    });
  }

  return <button onClick={publish}>Publish</button>;
}
```

The hook provides:

```ts
{
  supported: boolean;
  publish: (params) => Promise<void>;
}
```

This allows widget state or user selections to become available as model context where supported.

---

## View Tools

MCP Apps hosts can support tools registered directly by the mounted view.

Use `useViewTool()`:

```tsx
import { useViewTool } from "mcpfy-sdk/widget";

function Widget() {
  useViewTool(
    {
      name: "refresh_view",
      title: "Refresh View",
      description: "Refresh the current widget",
    },
    async () => {
      return {
        refreshed: true,
      };
    }
  );

  return <div>Widget</div>;
}
```

View tools are registered only when the current host supports them.

On hosts that do not support view tools, the registration becomes a no-op.

---

## Host Capabilities

Widgets can inspect capabilities through `useHostContext()`:

```tsx
const { capabilities } = useHostContext();
```

Capabilities allow a widget to determine whether features such as model context, display modes, or view tools are available.

This is preferable to assuming that every host supports every MCP widget feature.

---

## Widget Runtime

The `HostRuntime` component establishes the runtime context required by MCPfy widget hooks.

Basic structure:

```tsx
import {
  HostRuntime,
  ThemeProvider,
} from "mcpfy-sdk/widget";

export function App() {
  return (
    <ThemeProvider>
      <HostRuntime
        toolName="weather"
        appName="weather-widget"
        appVersion="1.0.0"
      >
        <WeatherWidget />
      </HostRuntime>
    </ThemeProvider>
  );
}
```

Hooks such as `useCallTool()`, `useHostContext()`, `useWidgetState()`, and `useModelContext()` must be used inside `HostRuntime`.

---

## Widget Lifecycle

At runtime, MCPfy performs the following general flow:

```text
MCP Tool Call
     │
     ▼
Widget-enabled Tool
     │
     ├── Structured Tool Result
     │
     └── Widget UI Content
             │
             ▼
        Host / MCP Client
             │
             ▼
        Widget Runtime
             │
             ▼
       React Widget
```

The React runtime establishes communication with the host and exposes that communication through hooks.

---

## Responsive Widget Sizing

MCPfy's React runtime observes changes to the widget document size and reports the dimensions to the parent host.

This allows compatible hosts to adjust the embedded widget's size as its content changes.

Widgets should therefore avoid unnecessary fixed-height assumptions and allow their content to size naturally where possible.

---

## Host Compatibility

A widget may run in different environments with different capabilities.

Do not assume that all hosts support:

* Widget state persistence
* Model-context updates
* Display-mode changes
* View-tool registration
* Host context metadata
* Native external-link handling

Use the runtime capabilities and protocol information to adapt behavior when necessary.

---

## Recommended Practices

### Keep the MCP tool useful without the UI

Always return meaningful structured data from the underlying tool.

```ts
return {
  city,
  temperatureC,
  condition,
};
```

This keeps the tool useful even when the client does not render the widget.

### Use the recommended tool-based widget API

Prefer:

```ts
server.tool({
  name: "weather",
  widget: "weather",
});
```

over the deprecated:

```ts
server.widget(...);
```

### Check host capabilities

Use:

```ts
const { capabilities } = useHostContext();
```

before relying on optional host features.

### Keep CSP restrictive

Only allow the external domains that the widget actually needs.

### Keep widget state serializable

Widget state is represented using:

```ts
Record<string, unknown>
```

Prefer simple serializable values such as strings, numbers, booleans, arrays, and objects.

### Handle loading and errors

Use the state exposed by `useCallTool()` or `useToolPayload()` to provide appropriate loading and error UI.

---

## Exports

Widget functionality is available through the `mcpfy-sdk/widget` entry point:

```ts
import {
  HostRuntime,
  ThemeProvider,
  useHostContext,
  useHostProtocol,
  useToolPayload,
  useCallTool,
  useSendFollowUp,
  useOpenExternal,
  useLayoutMode,
  useHostTheme,
  useLinkedTool,
  useWidgetState,
  useViewState,
  useModelContext,
  useViewTool,
  HostImage,
} from "mcpfy-sdk/widget";
```

The widget bridge functionality is available separately through:

```ts
import {
  // widget bridge APIs
} from "mcpfy-sdk/widget-bridge";
```

---

## Summary

MCPfy widgets provide a protocol-aware UI layer on top of standard MCP tools.

The main concepts are:

* **Tool + widget** — connects an MCP tool to an interactive UI.
* **Widget protocols** — supports MCP-UI, MCP Apps, and OpenAI Apps SDK.
* **Widget directory** — stores file-based widget implementations.
* **Widget runtime** — connects React widgets to their host.
* **Host context** — exposes protocol, layout, locale, platform, and capabilities.
* **Widget state** — allows compatible hosts to persist UI state.
* **Model context** — allows supported widgets to provide information back to the model.
* **View tools** — allows supported MCP Apps hosts to register tools from the mounted view.
* **Protocol abstraction** — lets the same widget adapt to different host environments.
