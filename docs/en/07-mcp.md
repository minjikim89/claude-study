# MCP (Model Context Protocol) — The Plug That Connects to the Outside World

## What Is MCP?

By default, Claude Code operates **within the local filesystem**. It can read and write files and run terminal commands, but it cannot directly access external services like Slack, Google Drive, or databases.

MCP (Model Context Protocol) solves this limitation. It is a **standardized external connection protocol** — a plug-in interface that gives Claude Code the ability to reach external services.

## Comparison with No-Code Tools

| No-code tool | MCP equivalent |
|---|---|
| **Zapier's App Connection** | MCP server connection (Slack, GitHub, etc.) |
| **Make's Module** | Individual tools provided by an MCP server |
| **n8n's Credential + Node** | MCP server configuration + tool call |
| **Notion's Integration** | MCP server connecting to the Notion API |

Key difference: no-code tools connect via **GUI clicks**; MCP connects by **registering a server address in a config file**.

## How It Works

```
┌─ Claude Code ─────────────────────────────────┐
│                                                │
│  "Send a Slack message"                        │
│       │                                        │
│       ▼                                        │
│  MCP Client (built-in)                         │
│       │                                        │
│       ▼                                        │
│  ┌─ MCP Server: Slack ────┐                    │
│  │  tool: send_message    │ ──▶ Slack API call  │
│  │  tool: read_channel    │                    │
│  │  tool: list_channels   │                    │
│  └────────────────────────┘                    │
│                                                │
│  ┌─ MCP Server: GitHub ───┐                    │
│  │  tool: create_issue    │ ──▶ GitHub API call │
│  │  tool: list_prs        │                    │
│  └────────────────────────┘                    │
│                                                │
│  ┌─ MCP Server: PostgreSQL┐                    │
│  │  tool: query           │ ──▶ Direct DB access│
│  │  tool: list_tables     │                    │
│  └────────────────────────┘                    │
└────────────────────────────────────────────────┘
```

**Structure summary:**
- **MCP Client**: built into Claude Code
- **MCP Server**: acts as a bridge between Claude Code and each external service
- **Tool**: an individual capability provided by an MCP server (e.g., send message, create issue)

## How to Configure MCP Servers

### Per-Project Configuration

Register in `.claude/mcp.json`:

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-..."
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

### Global Configuration

Register in `~/.claude/mcp.json` to make it available across all projects.

### Configuration Fields

| Field | Description | Example |
|---|---|---|
| `command` | Command to launch the MCP server | `npx`, `python`, `docker` |
| `args` | Launch arguments | Package name, config file path, etc. |
| `env` | Environment variables (API keys, etc.) | `SLACK_BOT_TOKEN`, `GITHUB_TOKEN` |

## Common MCP Servers

| Service | MCP server | Tools provided | Use case |
|---|---|---|---|
| **Slack** | @anthropic/mcp-server-slack | Send message, read channel | Daily report automation |
| **GitHub** | @anthropic/mcp-server-github | Create issue/PR, search code | PR review automation |
| **PostgreSQL** | @anthropic/mcp-server-postgres | SQL query, list tables | Data analysis reports |
| **Google Drive** | Community-provided | Read/write documents | Automated document generation |
| **Notion** | Community-provided | Page/DB manipulation | Project management sync |
| **Filesystem** | @anthropic/mcp-server-filesystem | Read/write files | Scoped directory access |

> Beyond the official Anthropic servers, many community-built MCP servers are available.

## MCP and Other Claude Code Features

```
Claude Code Tool Ecosystem
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Built-in tools   Read, Write, Bash, Glob, Grep, etc.
                 → Local filesystem operations

MCP tools        send_message, query, create_issue, etc.
                 → External service operations

Skill            Can define recipes that use MCP tools
                 → e.g., "Send weekly report to Slack every Monday" Skill

Subagent         Can create Subagents that use MCP tools
                 → e.g., DB query-only Subagent (only the query tool allowed)
```

## Skill + MCP Combination Example

Once MCP gives Claude access to external services, combining it with a Skill creates powerful automation workflows.

### Weekly Report Automation Skill

```markdown
# /weekly-report Skill

## Steps
1. Use GitHub MCP to fetch merged PRs from this week
2. Summarize the changes in each PR
3. Use Slack MCP to post a formatted report to the #team-updates channel

## Output Format
**[Weekly Dev Report]**
- Completed: N items
- In progress: N items
- Notable changes: ...
```

## Security Considerations

| Item | Best practice |
|---|---|
| **API key management** | Do not hardcode in `env`; prefer referencing environment variables |
| **Least privilege** | Grant only the minimum required permissions to the MCP server (read-only, etc.) |
| **Project vs. global** | Restrict sensitive connections to project-level configuration |
| **`.gitignore`** | Ensure config files containing API keys are excluded from git |

## Key Takeaways

- MCP = the standard protocol for **connecting Claude Code to external services**
- Equivalent to the "App Connection / Integration" concept in no-code tools
- Register a server in the config file (`mcp.json`) and Claude can use its tools
- Skill + MCP combination enables no-code-level automation workflows
- Manage API keys carefully and apply least-privilege principles
