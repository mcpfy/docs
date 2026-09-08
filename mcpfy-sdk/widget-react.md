
# MCPfy SDK Widget React

`mcpfy-sdk/widget` provides React components and hooks for building interactive MCP widgets that can run across supported MCP hosts.

The React runtime abstracts host-specific communication so the same widget can work with protocols such as **MCP Apps, ChatGPT Apps SDK, and MCP-UI** where supported.

---

## Installation

The React widget APIs are exported from the `mcpfy-sdk/widget` entry point.

```bash
npm install mcpfy-sdk react react-dom
```

`react` and `react-dom` are peer dependencies of the SDK and must be available in the application using the React widget package.

---

## Basic Structure

A widget normally uses `HostRuntime` as its root runtime provider:

```tsx
import { HostRuntime } from "mcpfy-sdk/widget";

export default function App() {
  return (
    <HostRuntime toolName="weather">
      <WeatherWidget />
    </HostRuntime>
  );
}
```

`HostRuntime` establishes the connection between the React application and the MCP host and makes the runtime available to the widget hooks.

---

# HostRuntime

```tsx
<HostRuntime
  toolName="weather"
  appName="weather-widget"
  appVersion="1.0.0"
>
  <WeatherWidget />
</HostRuntime>
```

### Props

| Property     | Type        | Description                                                   |
| ------------ | ----------- | ------------------------------------------------------------- |
| `toolName`   | `string`    | Name of the MCP tool associated with the mounted widget.      |
| `appName`    | `string`    | Optional application/widget name. Defaults to `mcpfy-widget`. |
| `appVersion` | `string`    | Optional application version. Defaults to `0.0.0`.            |
| `children`   | `ReactNode` | React content rendered inside the runtime.                    |

`HostRuntime` is required when using hooks that depend on the MCPfy widget runtime.

---

# ThemeProvider

`ThemeProvider` manages the widget theme and initially detects the user's preferred color scheme.

```tsx
import { ThemeProvider } from "mcpfy-sdk/widget";

export default function App() {
  return (
    <ThemeProvider>
      <HostRuntime toolName="weather">
        <WeatherWidget />
      </HostRuntime>
    </ThemeProvider>
  );
}
```

The provider exposes the current theme to the widget runtime.

Supported themes:

```ts
type HostTheme = "light" | "dark";
```

The theme is also written to:

```html
<html data-mcpfy-theme="light">
```

or:

```html
<html data-mcpfy-theme="dark">
```

---

# useHostContext

Returns information about the current MCP host.

```tsx
const host = useHostContext();

console.log(host.protocol);
console.log(host.layoutMode);
console.log(host.locale);
console.log(host.platform);
console.log(host.capabilities);
```

The returned value has the following shape:

```ts
interface HostEnv {
  protocol: HostProtocol;
  layoutMode: LayoutMode;
  locale?: string;
  platform?: string;
  capabilities: HostCapabilities;
}
```

This is useful when a widget needs to adapt its UI or behavior according to the host.

---

# useHostProtocol

Returns the protocol currently being used by the widget.

```tsx
const protocol = useHostProtocol();

if (protocol === "apps-sdk") {
  // ChatGPT Apps SDK environment
}
```

Possible values include:

* `apps-sdk`
* `mcp-apps`
* `mcp-ui`
* `iframe`
* `none`

The hook can also detect the environment when it is used outside an active runtime context.

---

# useToolPayload

Returns the input and output associated with the widget's MCP tool.

```tsx
const { input, output, isPending, error } = useToolPayload();
```

The returned structure is:

```ts
interface ToolPayload {
  input?: Record<string, unknown>;
  output?: Record<string, unknown>;
  isPending: boolean;
  error?: Error;
}
```

Example:

```tsx
function WeatherWidget() {
  const { input, output, isPending, error } = useToolPayload();

  if (isPending) {
    return <p>Loading...</p>;
  }

  if (error) {
    return <p>{error.message}</p>;
  }

  return (
    <div>
      <p>City: {String(input?.city ?? "")}</p>
      <p>Temperature: {String(output?.temperatureC ?? "")}°C</p>
    </div>
  );
}
```

---

# useCallTool

Calls another MCP tool from the widget.

There are two supported forms.

### Function form

```tsx
const callTool = useCallTool();

await callTool("get-weather", {
  city: "Delhi",
});
```

### Named-tool form

```tsx
const weather = useCallTool("get-weather");

await weather.call({
  city: "Delhi",
});
```

The named form returns:

```ts
interface CallToolHandle {
  call: (args?: Record<string, unknown>) => Promise<unknown>;
  isPending: boolean;
  data: Record<string, unknown> | undefined;
  error: Error | undefined;
}
```

The actual communication is delegated to the active host. Depending on the environment, mcpfy uses the available Apps SDK, MCP Apps, or host messaging mechanism.

---

# Typed Tool Calls

Widgets can optionally provide compile-time types for tools through module augmentation.

```tsx
declare module "mcpfy-sdk/widget" {
  interface WidgetToolMap {
    weather: {
      input: {
        city: string;
      };
      output: {
        city: string;
        temperatureC: number;
      };
    };
  }
}
```

The named form of `useCallTool` can then use those types:

```tsx
const weather = useCallTool("weather");

const result = await weather.call({
  city: "Delhi",
});

console.log(weather.data?.temperatureC);
```

This allows widget code to remain strongly typed without changing the runtime protocol.

---

# useLinkedTool

Returns the tool associated with the current mounted widget.

```tsx
const tool = useLinkedTool();

console.log(tool.name);

await tool.call({
  city: "Delhi",
});
```

The returned value is:

```ts
{
  name: string;
  call: (args?: Record<string, unknown>) => Promise<unknown>;
}
```

This is useful when the widget needs to call its own associated tool again.

---

# useSendFollowUp

Sends a follow-up message to the host/model.

```tsx
const sendFollowUp = useSendFollowUp();

await sendFollowUp(
  "Show me the weather forecast for tomorrow."
);
```

Depending on the host, mcpfy delegates the operation to the corresponding host API or messaging mechanism.

---

# useOpenExternal

Opens an external URL through the host.

```tsx
const openExternal = useOpenExternal();

openExternal("https://example.com");
```

When supported, the host controls how the external link is opened.

---

# useLayoutMode

Provides the current widget display mode and allows the widget to request a different mode.

```tsx
const layout = useLayoutMode();

console.log(layout.mode);
console.log(layout.available);
```

Request a new layout:

```tsx
await layout.request("fullscreen");
```

The returned object is:

```ts
{
  mode: LayoutMode;
  request: (mode: LayoutMode) => Promise<LayoutMode>;
  available: LayoutMode[];
}
```

The requested mode depends on the capabilities provided by the current host.

---

# useHostTheme

Returns the current widget theme.

```tsx
const theme = useHostTheme();

if (theme === "dark") {
  // Render dark-theme UI
}
```

Possible values:

```ts
"light" | "dark"
```

The hook considers both the active host runtime and the local theme provider.

---

# useWidgetState

Provides access to state that can be persisted by the host.

```tsx
const { state, setState } = useWidgetState();

await setState({
  selectedCity: "Delhi",
});
```

The returned value is:

```ts
{
  state: Record<string, unknown> | undefined;
  setState: (state: Record<string, unknown>) => Promise<void>;
}
```

When supported by the host, the state can survive widget updates or remounts.

---

# useViewState

`useViewState` combines local React state with host-persisted widget state and model context.

```tsx
const [state, setState] = useViewState({
  selectedCity: "Delhi",
  unit: "celsius",
});
```

Update the state:

```tsx
setState((previous) => ({
  ...previous,
  selectedCity: "Mumbai",
}));
```

When the state changes, mcpfy:

1. Updates the local React state.
2. Persists the state through the host when supported.
3. Publishes the state as model context when supported.

This makes `useViewState` useful for interactive widgets whose state should remain available to both the UI and the model.

---

# useModelContext

Provides access to model-context publishing.

```tsx
const modelContext = useModelContext();

console.log(modelContext.supported);
```

Publish context:

```tsx
await modelContext.publish({
  text: "The user selected Delhi.",
  structuredContent: {
    city: "Delhi",
  },
});
```

The returned value is:

```ts
{
  supported: boolean;
  publish: (params: ModelContextPublish) => Promise<void>;
}
```

`structuredContent` can be used when structured application state needs to be made available to the model.

---

# useViewTool

Registers a tool that can be called by the host/model while the widget is mounted.

```tsx
useViewTool(
  {
    name: "get-selected-city",
    title: "Get Selected City",
    description: "Returns the city currently selected in the widget.",
  },
  async () => {
    return {
      city: "Delhi",
    };
  }
);
```

The definition supports:

```ts
interface ViewToolDefinition<TInput = Record<string, unknown>> {
  name: string;
  title?: string;
  description?: string;
  schema?: z.ZodTypeAny;
}
```

The handler receives the tool arguments:

```tsx
useViewTool(
  {
    name: "search-city",
    description: "Search for a city.",
  },
  async (args) => {
    return {
      city: args.city,
    };
  }
);
```

View tools are supported where the host exposes the required capability. On unsupported environments, the registration becomes a no-op.

---

# Host Capabilities

Widgets can inspect host capabilities through `useHostContext()`:

```tsx
const { capabilities } = useHostContext();

if (capabilities.viewTools) {
  // View tools are supported
}
```

Capabilities are derived from the active protocol and host.

This allows widgets to progressively enhance their behavior rather than assuming that every MCP host supports every feature.

---

# HostImage

`HostImage` is a small image component that sets a safer default referrer policy for widget images.

```tsx
import { HostImage } from "mcpfy-sdk/widget";

export function Logo() {
  return (
    <HostImage
      src="https://example.com/logo.png"
      alt="Logo"
    />
  );
}
```

If the caller does not provide a `referrerPolicy`, `HostImage` defaults it to:

```text
no-referrer
```

A caller-provided `referrerPolicy` is preserved.

---

# Complete Example

```tsx
import {
  HostRuntime,
  ThemeProvider,
  useToolPayload,
  useCallTool,
  useHostContext,
  useHostTheme,
} from "mcpfy-sdk/widget";

function WeatherWidget() {
  const { output, isPending, error } = useToolPayload();
  const getWeather = useCallTool("weather");
  const { protocol } = useHostContext();
  const theme = useHostTheme();

  if (isPending) {
    return <p>Loading weather...</p>;
  }

  if (error) {
    return <p>Something went wrong: {error.message}</p>;
  }

  return (
    <div>
      <p>Protocol: {protocol}</p>
      <p>Theme: {theme}</p>
      <p>
        Temperature: {String(output?.temperatureC ?? "Unknown")}°C
      </p>

      <button
        onClick={() =>
          getWeather.call({
            city: "Delhi",
          })
        }
      >
        Refresh
      </button>
    </div>
  );
}

export default function App() {
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

---

# API Summary

| API               | Purpose                                                            |
| ----------------- | ------------------------------------------------------------------ |
| `HostRuntime`     | Provides the MCP host runtime to React widgets                     |
| `ThemeProvider`   | Manages light/dark widget themes                                   |
| `useHostContext`  | Accesses host protocol, layout, locale, platform, and capabilities |
| `useHostProtocol` | Detects the active host protocol                                   |
| `useToolPayload`  | Reads the current tool input/output state                          |
| `useCallTool`     | Calls an MCP tool                                                  |
| `useSendFollowUp` | Sends a follow-up message to the host/model                        |
| `useOpenExternal` | Opens an external URL through the host                             |
| `useLayoutMode`   | Reads and requests widget display modes                            |
| `useHostTheme`    | Reads the current theme                                            |
| `useLinkedTool`   | Calls the tool associated with the current widget                  |
| `useWidgetState`  | Reads and updates host-persisted widget state                      |
| `useViewState`    | Combines local state, persisted widget state, and model context    |
| `useModelContext` | Publishes context to the model                                     |
| `useViewTool`     | Registers a tool on a mounted widget view                          |
| `HostImage`       | Renders an image with a default `no-referrer` policy               |

---

# Best Practices

### Use `HostRuntime` at the widget root

Hooks such as `useToolPayload`, `useCallTool`, `useHostContext`, and `useViewState` depend on the runtime.

### Check capabilities before using optional features

```tsx
const { capabilities } = useHostContext();

if (capabilities.modelContext) {
  // Use model-context functionality
}
```

### Do not assume every host supports every API

mcpfy provides host adapters so widgets can work across multiple environments. Optional capabilities should therefore be detected rather than assumed.

### Keep widget state serializable

Widget state is represented as:

```ts
Record<string, unknown>
```

Prefer simple serializable values such as strings, numbers, booleans, arrays, and objects.

### Keep host-specific logic out of UI components

Prefer:

```tsx
const protocol = useHostProtocol();
```

over directly checking browser globals or implementing protocol-specific messaging inside individual components.

This keeps the widget portable across supported MCP hosts.