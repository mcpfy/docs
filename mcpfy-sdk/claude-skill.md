# Claude Skill

Build MCP servers with MCPfy using Claude.

The **MCPfy Server Builder** skill gives Claude the MCPfy-specific knowledge it needs to create, extend, debug, and verify MCP servers built with `mcpfy-sdk`.

## Overview

The skill helps Claude work with MCPfy's APIs, project structure, and development conventions so you can describe what you want to build without having to provide the MCPfy implementation details yourself.

It provides guidance for:

- MCP servers
- Tools
- Resources
- Prompts
- Widgets
- Authentication
- MCP clients
- Debugging and troubleshooting
- Server verification

The skill also includes MCPfy-specific reference material and a starter project template.

## What the Skill Does

### Build MCPfy servers

Ask Claude to create a new MCPfy server based on what you want to build.

For example:

```text
Create an MCPfy server with a tool that gets the current weather for a city.
````

The skill provides Claude with the MCPfy-specific instructions needed to structure and implement the server.

### Add tools

The skill helps Claude create and modify MCPfy tools using the SDK's conventions for schemas, handlers, and responses.

For example:

```text
Add a tool to my MCPfy server that searches GitHub repositories.
```

Claude can use the skill's guidance to implement the tool using the appropriate MCPfy APIs rather than falling back to lower-level MCP patterns.

### Build widgets

The skill includes guidance for MCPfy widgets and their expected project structure.

For example:

```text
Add a widget that displays the result of my weather tool.
```

MCPfy widgets follow the project's widget structure, for example:

```text
src/
└── widgets/
    └── weather/
        └── main.tsx
```

The skill helps Claude connect the widget to the corresponding tool and follow the MCPfy widget conventions.

### Work with resources and prompts

The skill also provides guidance for adding MCP resources and prompts to an existing MCPfy server.

For example:

```text
Add a resource that exposes my application's configuration.
```

or:

```text
Add a prompt that generates a code review request.
```

### Configure authentication

The skill includes MCPfy-specific guidance for authentication and helps Claude distinguish between the authentication patterns supported by the SDK.

For example:

```text
Add authentication to my MCPfy server.
```

Claude can use the skill's authentication references to determine the appropriate configuration for the project.

### Build MCPfy clients

The skill also covers the MCPfy client API, allowing Claude to help create clients that connect to MCP servers.

For example:

```text
Create an MCPfy client that connects to my server and calls the weather tool.
```

## MCPfy-Specific Guidance

The skill is designed to keep Claude aligned with MCPfy's SDK rather than generating generic MCP implementations.

For example, MCPfy tools use the SDK's `schema` convention:

```ts
schema: z.object({
  name: z.string(),
})
```

The skill also provides guidance for MCPfy response helpers such as:

```ts
text(...)
markdown(...)
object(...)
```

This helps generated code follow MCPfy's higher-level API instead of unnecessarily constructing lower-level MCP response objects.

## Verification

The skill encourages verifying generated implementations rather than assuming that code is correct after generation.

A typical workflow is:

```text
Create or modify the server
        ↓
Install dependencies
        ↓
Build the project
        ↓
Start the MCP server
        ↓
Connect with MCP Inspector
        ↓
Test tools
        ↓
Verify widgets and other capabilities
```

Use **MCP Inspector** to verify that the server can be connected to and that its capabilities behave as expected.

For example, after creating a tool, verify that:

1. The server starts successfully.
2. The tool appears in MCP Inspector.
3. The tool accepts the expected inputs.
4. The tool returns the expected result.

For widgets, also verify that the widget is discovered and renders correctly.

## Troubleshooting

The skill includes troubleshooting guidance for common MCPfy development issues.

This includes problems involving:

* TypeScript and build errors
* Incorrect MCPfy API usage
* Tool schemas
* Tool responses
* Widget structure and configuration
* Authentication configuration
* Server startup
* MCP connection issues

When an implementation does not work as expected, Claude can use the included troubleshooting references to identify likely causes and suggest fixes.

## What's Included

The MCPfy Server Builder skill contains:

* `SKILL.md` with the core MCPfy development instructions
* MCPfy server development references
* Tool development guidance
* Widget development guidance
* Authentication guidance
* Client guidance
* Troubleshooting references
* A starter project template

The references are designed to give Claude the information it needs without requiring the entire MCPfy documentation to be included in every conversation.

## Getting Started

Install the **MCPfy Server Builder** skill in Claude, then describe the MCPfy server or feature you want to build.

For example:

```text
Create an MCPfy server with a weather tool and a widget that displays the weather result.
```

You can also use the skill with an existing MCPfy project:

```text
Add a GitHub repository search tool to my existing MCPfy server.
```

or:

```text
Debug this MCPfy widget and make sure it works with the server.
```

Claude can then use the MCPfy-specific skill instructions while working on the project.

## Keep the Skill Updated

The skill is based on MCPfy SDK APIs and project conventions.

When MCPfy introduces changes to its APIs or recommended project structure, the skill should be updated accordingly so that Claude continues to generate compatible MCPfy projects.
