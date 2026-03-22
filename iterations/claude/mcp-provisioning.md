# MCP Server Provisioning Guide

This document defines the rules and decision framework you must follow when establishing, configuring, or modifying MCP (Model Context Protocol) connections for the user.

## Core Principle: User-Level, Lazy-Loaded by Default

The user works independently — not on collaborative projects involving shared MCP configurations. Never assume federated authentication or shared access models.

**Default behavior**: All MCP connections should be provisioned at the **user level** so they are available on-demand regardless of the active project or working directory. This eliminates the maintenance burden of per-repo MCP definitions.

**Exception**: If an MCP is clearly project-specific (e.g., a repo-scoped database tool), create a `.mcp.json` in the project root. MCP configurations are additive, so project-level definitions layer on top of user-level ones without conflict.

## Decision Matrix: Where to Create an MCP Connection

Use this tiered preference system when deciding where to provision a new MCP:

### Tier 1 — Claude.ai Server-Side Connectors (Preferred)

Anthropic supports MCP connections created via Claude.ai. These are:
- Managed server-side by Anthropic
- Inherited automatically by Claude Code
- Lazy-loaded to prevent context bloat
- Persistently available across all contexts

**Use this tier** for any generally-useful MCP that doesn't require local filesystem access or multi-context switching.

#### When Claude.ai Connectors Are Not Suitable

**Multi-context identity switching**: The user uses Claude Code for both personal and business purposes (self-employed consultant). Some MCP tools (e.g., Gmail, Google Drive) need context-specific configurations:

- *Business context*: Gmail MCP authenticated against the business Workspace → client emails sent from the business address
- *Personal context*: Gmail MCP authenticated against the personal Workspace → personal correspondence

Claude.ai does not currently support multiple account configurations per connector. Workarounds:
1. Create two local MCP instances (one per identity)
2. Create a primary connection in Claude.ai and a secondary local one

**Custom MCP servers**: The user frequently builds custom MCP servers tailored to specific workflows, often with hardcoded preferences (e.g., `fal-nano-mcp` on NPM with preset AI model selections). These are published as NPM packages for portability and open-sourcing. They cannot be deployed as Claude.ai connectors unless hosted as SSE endpoints.

### Tier 2 — Self-Hosted Aggregator (MetaMCP)

The user runs a MetaMCP instance as a self-managed MCP aggregator in a managed environment.

**Current utility**:
- Hosting environment for custom MCP servers
- Tool-agnostic aggregation layer (not bound to any specific AI agent)

**Use this tier** for custom-built MCPs that don't require local filesystem access. The advantage over Claude.ai connectors is vendor independence — connections provisioned here work with any MCP-compatible AI tool, not just Claude.

### Tier 3 — Local (Localhost)

**Use this tier** when the MCP requires direct access to the local filesystem.

**Why this matters**: MCPs that read or process local binary files (e.g., a transcription MCP sending a local audio file to a Google API) cannot reliably run remotely. While proxy mechanisms exist for tunneling local files to remote MCPs, the tooling is immature and fragile.

Contrast with generative AI MCPs where the API returns a download link — no local file access is needed, so Tier 1 or 2 is appropriate.

**Trade-off**: Localhost MCPs must be configured per-machine, losing the portability of remotely managed connections. Accept this cost only when local file access is genuinely required.

## MCP Configuration Format

When creating MCP connections for Claude Code at the user level, write them to `~/.claude/settings.json` under the `mcpServers` key. Each entry follows this schema:

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "API_KEY": "value"
      }
    }
  }
}
```

For SSE-based remote servers (including MetaMCP-hosted ones):

```json
{
  "mcpServers": {
    "server-name": {
      "type": "sse",
      "url": "https://example.com/sse-endpoint"
    }
  }
}
```

### Naming Conventions

- Use lowercase kebab-case for server names: `google-drive`, `fal-nano`, `hf-hub`
- Name should reflect the service or capability, not the package name
- For multi-context servers, suffix with the context: `gmail-business`, `gmail-personal`

### Environment Variables and Secrets

- Never hardcode secrets directly into committed files
- For user-level configs, secrets in `~/.claude/settings.json` are acceptable (it's gitignored by design)
- For project-level `.mcp.json`, reference environment variables rather than embedding keys
- If an API key is needed, check if the user already has it set in their environment before asking

## Logging and Observability

When setting up or debugging MCP connections, use structured logging:

```
[MCP:server-name] action: description
[MCP:server-name] status: connected | failed | timeout
[MCP:server-name] error: error message
```

After provisioning a new MCP, verify the connection by invoking a lightweight tool call (e.g., a list or status operation) and report the result.

## Summary Decision Flowchart

```
Is the MCP generally useful across projects?
├── YES → Does it need local filesystem access?
│   ├── YES → Tier 3 (localhost)
│   └── NO → Is it a custom/self-built MCP?
│       ├── YES → Tier 2 (MetaMCP) or Tier 3 if filesystem needed
│       └── NO → Tier 1 (Claude.ai connector)
└── NO → Create project-level .mcp.json
```
