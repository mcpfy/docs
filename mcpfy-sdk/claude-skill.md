# Claude Skill

Build MCP servers with MCPfy using Claude.

The **MCPfy Server Builder** is an agent skill that gives Claude the MCPfy-specific knowledge and patterns needed to build MCP servers with `mcpfy-sdk`.

## Install

Install the MCPfy Server Builder skill with:

```bash
npx skills add mcpfyy/mcpfy --skill mcpfy-server-builder
```

Once installed, Claude can use the skill when creating or modifying MCPfy projects.

## Use the Skill

After installing the skill, describe what you want Claude to build.

```text
Create an MCPfy server with a weather tool and a widget that displays the weather result.
```

Claude can use the skill to build the server using MCPfy's SDK conventions.

For example:

```text
Create an MCPfy server with a tool that searches GitHub repositories.
```

```text
Add a widget that displays the result of my weather tool.
```

```text
Add authentication to my MCPfy server.
```

```text
Create an MCPfy client that connects to my server and calls the weather tool.
```

You can also use it with an existing MCPfy project:

```text
Add a GitHub repository search tool to my existing MCPfy server.
```

## What It Supports

The MCPfy Server Builder skill provides guidance for:

* **Servers** — creating and structuring MCPfy servers
* **Tools** — MCPfy schemas, handlers, and response helpers
* **Resources** — exposing application data through MCP resources
* **Prompts** — creating MCPfy prompts
* **Widgets** — building and integrating MCPfy widgets
* **Authentication** — configuring supported authentication patterns
* **Clients** — connecting to MCP servers with the MCPfy client
* **Debugging** — diagnosing common MCPfy development issues
* **Verification** — building and testing MCPfy implementations

The skill uses MCPfy-specific references and a starter project template so Claude can work with the SDK's actual APIs and conventions.

## Example

After installing the skill, you can simply tell Claude:

```text
Build an MCPfy server for a weather service. It should have a weather tool and a widget that displays the weather data.
```

Claude can then use the MCPfy Server Builder skill to determine the appropriate project structure and MCPfy APIs.

## Updating the Skill

The skill is maintained alongside the MCPfy SDK.

When MCPfy APIs or recommended development patterns change, the skill should be updated to keep generated projects compatible with the latest SDK.
