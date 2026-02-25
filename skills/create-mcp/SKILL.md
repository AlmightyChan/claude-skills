---
name: create-mcp
description: Configure MCP servers to connect Claude Code to external tools. Proactively use when adding, configuring, or managing MCP servers, tools, or integrations.
user-invocable: false
version: 0.1.0
---

# Configuring an MCP Server

When adding an MCP server, follow this complete process:

## 0. Research Official Documentation
Before proceeding, WebFetch the latest official docs:
- MCP servers: https://code.claude.com/docs/en/mcp

Apply any patterns, fields, or conventions from the official docs.
If the docs describe features not covered below, prefer the official docs.

## 1. Identify the Server Type
| Transport | Use case | Example |
|-----------|----------|---------|
| HTTP | Cloud services (recommended) | `https://mcp.notion.com/mcp` |
| SSE | Legacy remote servers | `https://mcp.asana.com/sse` |
| Stdio | Local processes/scripts | `npx -y airtable-mcp-server` |

## 2. Choose the Scope
| Scope | Flag | Storage | Use case |
|-------|------|---------|----------|
| `local` | (default) | `~/.claude.json` | Personal, project-specific |
| `project` | `--scope project` | `.mcp.json` | Team-shared (version controlled) |
| `user` | `--scope user` | `~/.claude.json` | Available across all projects |

## 3. Add the Server

### HTTP (Recommended for remote)
```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http api https://api.example.com/mcp \
  --header "Authorization: Bearer token"
```

### Stdio (Local processes)
```bash
claude mcp add --transport stdio --env KEY=value <name> -- <command> [args]
claude mcp add --transport stdio --env AIRTABLE_API_KEY=KEY airtable \
  -- npx -y airtable-mcp-server
```

## 4. Authenticate if Required
For OAuth-enabled servers:
1. Run `/mcp` in Claude Code
2. Select the server requiring authentication
3. Follow browser prompts to complete OAuth flow

## 5. Verify the Connection
```bash
claude mcp list              # List all servers
claude mcp get <name>        # Get server details
/mcp                         # Check status in Claude Code
```

## 6. Use the Server
- **Resources**: Reference with `@server:protocol://path`
- **Prompts**: Invoke as `/mcp__server__prompt`
- **Tools**: Available automatically to Claude

---

# Reference

## Managing Servers
```bash
claude mcp list              # List all servers
claude mcp get <name>        # Get server details
claude mcp remove <name>     # Remove a server
claude mcp add-json <name> '<json>'  # Add from JSON config
claude mcp add-from-claude-desktop   # Import from Claude Desktop
```

## .mcp.json Format (Project Scope)

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    },
    "local-tool": {
      "command": "npx",
      "args": ["-y", "some-package"],
      "env": {
        "KEY": "${MY_KEY}"
      }
    }
  }
}
```

## Environment Variable Expansion
- `${VAR}` - Expands to value of VAR
- `${VAR:-default}` - Uses default if VAR not set

## Plugin MCP Servers
Plugins can bundle MCP servers in `.mcp.json` or inline in `plugin.json`. These start automatically when the plugin is enabled.

**Documentation**: https://code.claude.com/docs/en/mcp

## Completion Token
When this skill completes successfully, output: `[SKILL_COMPLETE:create-mcp]`
