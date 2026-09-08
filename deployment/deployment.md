# MCP Server Deployment Using GitHub and Mcpfy

This repository documents the process of deploying an MCP (Model Context Protocol) server using **Mcpfy**, with source code hosted on **GitHub**.

## Overview

The deployment workflow:

```
Local MCP Server
      ↓
Git Repository
      ↓
Connect GitHub to Mcpfy
      ↓
Select Repository & Branch
      ↓
Configure Build / Runtime
      ↓
Deploy
      ↓
Mcpfy Hosted MCP Endpoint
      ↓
Test using MCP Inspector
```

Mcpfy provides GitHub-based deployment for MCP servers. A repository can be connected to the platform, configured with the appropriate build/runtime settings, and deployed as a hosted MCP endpoint. Mcpfy also supports automatic deployments when changes are pushed to the connected repository.

## Prerequisites

- A Mcpfy account
- A GitHub account
- An MCP server project
- The MCP server code pushed to GitHub
- A valid Git branch containing the code
- Any required environment variables or secrets

**Repository used:** `thepandeyakash/Mcp`
**MCP TypeScript server location:** `mcp-typescript/`

## Project Structure

```
Mcp/
└── mcp-typescript/
    ├── src/
    │   ├── index.ts
    │   ├── client.ts
    │   └── ai-client.ts
    ├── package.json
    ├── package-lock.json
    ├── tsconfig.json
    └── .env
```

Main server implementation: `mcp-typescript/src/index.ts`

### Exposed Tools

| Tool | Description |
|------|-------------|
| `add` | Adds two numbers |
| `multiply` | Multiplies two numbers |
| `greet` | Generates a greeting for a person |

### Exposed Resource

- `info://server` — returns information about the server

## Deploying to Mcpfy

### 1. Connect GitHub to Mcpfy

1. Open the Mcpfy dashboard.
2. Navigate to **Servers**.
3. Select **New Server**.
4. Select **Import from GitHub**.
5. Connect your GitHub account.
6. Authorize Mcpfy to access the required repositories.
7. Select the repository containing the MCP server.

### 2. Select the Repository

Since `Mcp` is a multi-directory repository (not one where the MCP server lives at the root), the repository root directory must be configured explicitly.

```
Mcp/
├── mcp-typescript/
└── mcpfy-demo/
```

### 3. Deployment Configuration

| Configuration | Value |
|---|---|
| GitHub Repository | `thepandeyakash/Mcp` |
| Branch | `main` |
| Preset | MCP TypeScript SDK |
| Repository Root Directory | `mcp-typescript` |
| Region | `us-central-1` |
| Port | `3000` |

The **MCP TypeScript SDK** preset is used because the server is implemented in TypeScript with the MCP TypeScript SDK. Mcpfy applies framework-specific presets (e.g., MCP TypeScript SDK, MCP Python SDK) to configure the deployment appropriately.

> **Repository Root Directory** tells Mcpfy to treat `mcp-typescript` as the application directory instead of the repo root — essential for monorepos with multiple projects.

### 4. Build Configuration

```bash
npm install && npx tsc
```

- `npm install` — installs dependencies from `package.json`
- `npx tsc` — compiles the TypeScript source using `tsconfig.json`

```
package.json → npm install → dependencies installed → npx tsc → compiled
```

### 5. Environment Variables

Sensitive values should be configured through Mcpfy's dashboard rather than committed to the repository.

| Key | Value |
|---|---|
| `API_KEY` | `********` |

> No additional runtime secrets were required for the core `add`, `multiply`, and `greet` functionality.

### 6. Deploy

1. Verify the repository.
2. Verify the branch.
3. Verify the repository root directory.
4. Verify the build configuration.
5. Review environment variables.
6. Click **Deploy**.

The deployment page shows status, branch, commit, deployment URL, and logs. A successful deployment shows status: `deployed`.

## Deployment URL

Mcpfy generates a hosted MCP endpoint following the pattern:

```
https://<deployment-url>/mcp
```

The `/mcp` endpoint is an MCP protocol endpoint, not a conventional webpage. Opening it directly in a browser may show **"Method Not Allowed"** — this does not indicate a failed deployment. Use Mcpfy's **Inspector** or an MCP-compatible client instead.

## Testing the Deployed Server

Use **Mcpfy Inspector** to:

- Connect to deployed MCP servers
- Call tools
- Inspect JSON-RPC communication
- Browse resources
- Replay requests

### Testing Tools

**`add`**
```json
Input:  { "a": 2, "b": 3 }
Output: 5
```

**`multiply`**
```json
Input:  { "a": 4, "b": 5 }
Output: 20
```

**`greet`**
```json
Input:  { "name": "Akash", "formal": false }
Output: "Hey Akash!"

Input:  { "name": "Akash", "formal": true }
Output: "Good day, Akash."
```

### Testing the Resource

**`info://server`**
```
Output: "This is Akash's first MCP server."
```

## Git-Based Deployment Workflow

```
Developer changes code
        ↓
git add
        ↓
git commit
        ↓
git push
        ↓
GitHub
        ↓
Mcpfy detects repository update
        ↓
Build
        ↓
Deploy
        ↓
Updated MCP server
```

Once configured, pushes to the connected branch automatically trigger new deployments — no need to manually redeploy each time.

### Example Workflow

```bash
git status
git add .
git commit -m "Update MCP server"
git push origin main
```

## Deployment Verification

| Level | What to check |
|---|---|
| 1. Deployment Status | Shows `deployed` |
| 2. Deployment Logs | No build or runtime errors |
| 3. MCP Endpoint | Endpoint exists and is reachable |
| 4. MCP Inspector | Connection, tool discovery, tool execution, resource discovery, returned results |

Level 4 (Inspector) is the most meaningful functional verification for an MCP server.

## Troubleshooting

**Deployment fails during build**
- Check build command
- Check repository root directory (`mcp-typescript`)
- Check branch
- Check `package.json` / `tsconfig.json`
- Check dependency installation
- Review deployment logs

**Files cannot be found during deployment**
- Verify the repository root points to `mcp-typescript`, not the repo root

**Browser shows "Method Not Allowed"**
- Expected — don't test `/mcp` in a browser. Use Inspector or an MCP client instead.

**Tools are not available**
- Check deployment logs
- Verify the correct source directory is deployed
- Verify the MCP server starts correctly
- Connect through Inspector and check registered tools

## Deployment Architecture

```
                 GitHub
                   │
                   │ push
                   ▼
          ┌─────────────────┐
          │     Mcpfy       │
          │ Git Integration │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ Build & Deploy  │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │ Hosted MCP      │
          │ Server          │
          └────────┬────────┘
                   │
                   │ MCP
                   ▼
          ┌─────────────────┐
          │ MCP Inspector / │
          │ MCP Client      │
          └─────────────────┘
```

## Key Learnings

- **Git-based deployment** — the GitHub repository is the source of truth for the deployed application.
- **Repository root configuration** — for monorepos, the platform needs to know which directory contains the app.
- **Build process** — dependencies are installed and TypeScript is compiled before the server runs.
- **Environment configuration** — secrets should live in the deployment environment, not the repo.
- **MCP endpoint** — not a conventional webpage; test with an MCP-aware client.
- **Inspector** — used to test tools and inspect MCP protocol traffic.
- **Git → Deployment lifecycle** — commits and pushes flow through build/deploy to an updated hosted server.

## Conclusion

```
Develop MCP server
       ↓
Push code to GitHub
       ↓
Connect GitHub to Mcpfy
       ↓
Select repository and branch
       ↓
Configure repository root
       ↓
Configure build/runtime
       ↓
Deploy
       ↓
Obtain MCP endpoint
       ↓
Test using Inspector
       ↓
Push future changes
       ↓
Redeploy updated server
```

This workflow provides a Git-based deployment model for MCP servers, where source-code changes move through the same repository-driven process used for modern application deployments. Mcpfy also provides deployment logs, environment configuration, branch previews, and MCP-specific inspection capabilities as part of its platform.