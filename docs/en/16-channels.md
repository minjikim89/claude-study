# Channels — How External Systems Talk to Claude

## What Are Channels?

Until now, Claude Code always required **you to initiate the conversation**. You sit at the terminal, send a message, Claude responds.

Channels reverse this direction. **External systems (Telegram, Discord, CI pipelines, etc.) push events to Claude, and Claude processes them and replies back through the same channel**.

You don't need to be at a terminal. When a Telegram message arrives, the Claude running on your computer reads it, opens local files, does the work, and sends the result back to Telegram.

> Summary: Channels = **the interface through which the outside world talks to Claude**

## Comparison with No-Code Tools

| No-Code Tool | Channels Equivalent |
|---|---|
| **Zapier Webhook Trigger** | External event arrives → workflow starts |
| **Make Webhook module** | Receives CI events, monitoring alerts and processes them |
| **Slack Bot** | Send a message in chat → bot responds |
| **Discord Bot** | Two-way conversation with a bot in a server/DM |
| **n8n Webhook Node** | Receives HTTP requests and starts automation |

Key differences:
- No-code bots run **in the cloud** → no access to local files
- Channels runs **your Claude on your computer** → local files, Git, and MCP servers all available
- No-code tools execute **only the configured automation** → Channels lets **Claude use judgment** to handle the situation

## How It Works

```
┌─ External ───────────────────────────────────────────┐
│                                                      │
│  Telegram                Discord               CI    │
│  phone message           DM / server           alert │
│      │                      │                   │    │
└──────┼──────────────────────┼───────────────────┼────┘
       │                      │                   │
       ▼                      ▼                   ▼
┌─ Channel Plugin (MCP server) ─────────────────────────┐
│                                                        │
│  Receives message/event → converts to <channel> event  │
│                                                        │
└────────────────────────┬───────────────────────────────┘
                         │ push
                         ▼
┌─ Your Computer — Claude Code session (running) ────────┐
│                                                        │
│  Receives <channel source="telegram"> event            │
│  → Claude reads and makes a decision                   │
│  → Works with local files / Git / MCP servers          │
│  → Replies to the same channel using the reply tool    │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Core structure**:
- Channel Plugins are **a special form of MCP server** — but unlike regular MCP, events are **pushed** to Claude rather than Claude pulling data
- Events only arrive **while a session is open** (session must stay active in the background)
- Terminal output: incoming messages are visible, but **replies go directly to the channel — not shown in the terminal**

## Channels vs. Remote Control vs. Scheduled Tasks

All three feel like "controlling Claude from outside the terminal," but **who initiates the action is completely different**.

| | **Remote Control** | **Channels** | **Scheduled Tasks** |
|---|---|---|---|
| **Who acts** | You control the session directly from your phone | External system pushes an event to Claude | Claude runs automatically when the time comes |
| **Trigger** | You type a message | External event arrives (message, alert, webhook) | A scheduled time or interval |
| **Flow direction** | You → Claude | External → Claude → External | Claude auto-runs |
| **Best for** | Continuing and controlling in-progress work | CI alert handling, bot responses, remote commands | Scheduled reports, periodic monitoring |
| **Analogy** | TV remote | Doorbell | Alarm clock |

> **Quick rule**:
> "I want to issue commands from my phone" → Remote Control
> "I want an external system to notify Claude and have it process things" → Channels
> "I want it to run automatically at a set time" → Scheduled Tasks

## Channels vs. Regular MCP

Both use the Plugin/MCP structure, but they work differently.

| | **Regular MCP Server** | **Channel** |
|---|---|---|
| **Direction** | Claude pulls when needed | External pushes to Claude |
| **Trigger** | Claude during a task: "get data from DB" | External message/event arrives |
| **Example** | Notion MCP, Slack MCP (for querying) | Telegram Channel, Discord Channel |
| **Analogy** | Librarian (retrieves on request) | Mail carrier (rings the bell when a letter arrives) |

## Supported Channels (Research Preview)

Currently supported channels:

| Channel | Plugin Name | Use Case |
|---|---|---|
| **Telegram** | `telegram@claude-plugins-official` | Bot DMs, group messages |
| **Discord** | `discord@claude-plugins-official` | DMs, server channels |
| **fakechat** | `fakechat@claude-plugins-official` | localhost demo (for testing) |

> `fakechat` is an official demo channel testable directly in a browser (`http://localhost:8787`). Useful for experiencing Channel behavior without a real app account.

## Setup — Telegram Walkthrough

### Prerequisites

```bash
# Bun is required (Channel Plugins are Bun scripts)
bun --version        # install if missing
```

Bun installation: https://bun.sh/docs/installation

### Step 1: Install the Plugin

```
/plugin install telegram@claude-plugins-official
```

If this is your first time, add the Marketplace first:
```
/plugin marketplace add anthropics/claude-plugins-official
```

### Step 2: Set the Bot Token

Get your token from [BotFather](https://t.me/BotFather) via `/newbot`, then run:

```
/telegram:configure <token>
```

→ Saved to `~/.claude/channels/telegram/.env`

### Step 3: Enable the Channel and Run

```bash
claude --channels plugin:telegram@claude-plugins-official
```

### Step 4: Pair (First Run Only)

1. Send any message to your bot on Telegram
2. The bot returns a pairing code
3. Enter in Claude Code:

```
/telegram:access pair <code>
/telegram:access policy allowlist    ← restrict to your account (security)
```

### Discord Setup

The structure is identical. Create a bot at the [Discord Developer Portal](https://discord.com/developers/applications) → enable Message Content Intent → invite to server → configure token.

```
/plugin install discord@claude-plugins-official
/discord:configure <token>
claude --channels plugin:discord@claude-plugins-official
```

## Security Model

Every Channel Plugin maintains a **sender allowlist**.

```
Message from an account not on the allowlist
        → silently ignored (no error)

Message from an account on the allowlist
        → delivered to Claude Code as a channel event
```

After pairing, setting `/telegram:access policy allowlist` ensures only **registered accounts** can reach Claude.

Additionally:
- The `--channels` flag is required to activate a Channel (`.mcp.json` alone won't receive messages)
- Team/Enterprise plans require an admin to enable `channelsEnabled`

## Real-World Scenarios

### 1. Remote Commands from Your Phone
```
[Situation] You're out. Claude Code is running on your home computer.

You send via Telegram: "What files did I change today and what's the git status?"

→ Claude on your computer runs git commands
→ Sends the result back to Telegram
→ You see it immediately on your phone
```

### 2. CI Failure Alert → Instant Analysis
```
[Situation] You have a Claude Code session open with CI running in the background.

Webhook → Channels: "Build failed: TypeError in auth.ts line 42"

→ Claude opens the file and analyzes the error
→ Replies on Discord: "Undefined reference on line 42 of auth.ts. Want me to commit a fix?"
→ You reply "yes" from your phone → Claude edits the file and commits
```

### 3. Monitor and Intervene in Long-Running Work
```
[Situation] Starting a large refactor, expected to take 1–2 hours.

Run with: claude --channels plugin:telegram@claude-plugins-official

Midway: Telegram: "How far along are you?"
→ Claude: "src/api/ done, working on src/db/ (43%)"

Telegram: "Don't touch db/legacy.ts"
→ Claude immediately adjusts course
```

## Requirements

| Item | Requirement |
|---|---|
| Claude Code version | v2.1.80 or later |
| Authentication | claude.ai login (Console/API key not supported) |
| Dependency | Bun must be installed |
| Plan | Pro/Max (personal), Enterprise requires admin activation |
| Status | Research preview (opt-in via `--channels` flag) |

## Relationship to Previous Concepts

```
Channels = "push-direction MCP" + "event-driven automation"

Related concepts:
  ├── MCP (07)        ← Channel is a special form of MCP (but push direction)
  ├── Plugins (13)    ← Channels are installed and managed as Plugins
  ├── Hooks (08)      ← Hooks = internal Claude Code events / Channels = external events
  └── Remote Control (15) ← Similar surface, but opposite direction
```

## Key Takeaways

- Channels = external events **pushed into a running session** (the push version of MCP)
- Connect a Telegram or Discord bot → **that chat window becomes Claude's input/output interface**
- Runs on your machine → **local files, Git, and MCP servers are all available**
- Unlike Remote Control (you control Claude), Channels = **external systems talk to Claude**
- Security: allowlist-based — messages from unregistered accounts are silently ignored
- Opt-in per session with `--channels` flag; `Bun` required; **v2.1.80+ required**
- Research preview — `--channels` flag syntax and protocol may change
