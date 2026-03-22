# How-To-MCP

A guide for instructing AI agents (like Claude Code) on how to provision and manage MCP (Model Context Protocol) server connections according to a user's specific preferences.

## Repository Structure

- **`iterations/raw.md`** — The original hand-drafted instructions. Useful as a reference for the thought process behind the provisioning rules, but unstructured and informal.

- **`iterations/claude/mcp-provisioning.md`** — An improved, structured version of the same instructions, organized for direct use by an AI agent. Contains a tiered decision matrix, configuration format examples, naming conventions, and a decision flowchart.

**Recommendation**: Use the `iterations/claude/` version. It contains the same logic as `raw.md` but is structured for machine consumption.

## Background

MCP connections can be provisioned at multiple levels — cloud-managed (Claude.ai connectors), self-hosted aggregators (like MetaMCP), or localhost. Each has trade-offs around portability, filesystem access, and vendor independence. This guide encodes a specific preference hierarchy so that an AI agent can make the right choice autonomously when asked to "set up an MCP for X."

---

## How to Use This in Your Own Setup

Claude Code supports a `CLAUDE.md` file in `~/.claude/` that acts as persistent instructions loaded into every conversation. Rather than stuffing all instructions into one large file, the recommended pattern is to keep `CLAUDE.md` concise and reference detailed topic-specific documents stored in a subdirectory.

### Directory Structure

```
~/.claude/
├── CLAUDE.md                  # Main instruction file (always loaded)
├── context/                   # Detailed reference documents (loaded on demand)
│   ├── mcp-provisioning.md    # How to CREATE new MCP connections
│   ├── system-environment.md  # OS, hardware, network details
│   ├── development-environment.md
│   └── ...other context files
└── settings.json              # Claude Code settings (includes mcpServers definitions)
```

### Step-by-Step

#### 1. Create the Context Directory

```bash
mkdir -p ~/.claude/context
```

#### 2. Install the MCP Provisioning Instructions

Copy or adapt the provisioning guide from this repository:

```bash
cp iterations/claude/mcp-provisioning.md ~/.claude/context/mcp-provisioning.md
```

Edit the file to reflect your own preferences:
- Which tier is your default? (Claude.ai connectors, a self-hosted aggregator, or localhost?)
- Do you have multi-context needs (personal vs. work identities)?
- Do you build custom MCP servers?
- What is your aggregator URL, if any?

#### 3. Reference It from CLAUDE.md

Add an entry to your `~/.claude/CLAUDE.md` pointing the agent to the context file:

```markdown
## Context Directory

Detailed context is split into files under `~/.claude/context/`. Read the relevant file(s) when needed:

| File | Contents |
|------|----------|
| `mcp-provisioning.md` | Rules for creating/configuring new MCP connections (tiers, formats, naming) |
| ...other files... |
```

The key here is that `CLAUDE.md` is always loaded, but the context files are only read when relevant. This keeps context lean while making detailed instructions available on demand.

#### 4. Configure the MCP Servers Themselves

MCP server definitions live in `~/.claude/settings.json`. Example:

```json
{
  "mcpServers": {
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@context7/mcp"]
    },
    "huggingface": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@huggingface/mcp-server"],
      "env": {
        "HF_TOKEN": "hf_your_token_here"
      }
    },
    "metamcp": {
      "type": "sse",
      "url": "https://your-metamcp-instance.example.com/sse"
    }
  }
}
```

### How the AI Agent Should Handle MCP Setup Requests

With the provisioning guide in place, when you say "set up an MCP for X," the agent should:

1. **Read the provisioning guide** from `~/.claude/context/mcp-provisioning.md`
2. **Determine the tier** using the decision flowchart in that document
3. **Write the configuration** to the appropriate location (`settings.json` for user-level, `.mcp.json` for project-level)
4. **Follow naming conventions**: lowercase kebab-case, descriptive of the service
5. **Handle secrets safely**: keys in `settings.json` are fine (not committed); project-level configs should reference env vars
6. **Verify the connection** with a lightweight tool call
7. **Log the result** in structured format:
   ```
   [MCP:server-name] action: provisioned at user level (Tier 3, localhost)
   [MCP:server-name] status: connected
   [MCP:server-name] tools: list-files, read-file, write-file (3 tools available)
   ```

### Customizing for Your Workflow

The tiered preference system reflects one user's workflow. Adapt it:

- **No self-hosted aggregator?** Remove Tier 2. Your decision simplifies to: Claude.ai connector vs. localhost.
- **Collaborative projects?** Add guidance about project-level `.mcp.json` and shared authentication.
- **Multiple AI agents?** Emphasize Tier 2 (aggregator) for vendor independence.
- **Primarily need filesystem access?** Default to Tier 3 and note the portability trade-off.
