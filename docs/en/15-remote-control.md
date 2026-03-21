# Remote Control — Continue Your Claude Code Session from Your Phone

## What Is Remote Control?

When you're working in Claude Code at your desk and need to step away — you can continue **that exact session** from your phone on the couch, or from a browser on another computer.

The key is that everything keeps **running on your machine**. Your code, files, and MCP servers all stay in your local environment. Your phone or browser acts only as **a window into that running session**.

> Summary: Remote Control = **a remote for your terminal**

## Comparison with No-Code Tools

| No-Code Tool | Remote Control Equivalent |
|---|---|
| **Zapier mobile app** | View execution results on your phone |
| **Notion mobile app** | Continue editing on your phone what you started on desktop |
| **Google Docs mobile** | Access the same document from anywhere |
| **TeamViewer / Remote Desktop** | Control your computer from another device |

Key difference:
- No-code apps access **data stored in the cloud** → same from anywhere
- Remote Control accesses a **session running on your computer** → local files and MCP servers work as-is

## How It Works

```
┌─ Your Computer ───────────────────────────────┐
│                                                │
│  Claude Code session (running locally)         │
│  ├── Your file system                          │
│  ├── MCP servers (Slack, DB, etc.)             │
│  ├── Plugins/Skills                            │
│  └── Project settings                          │
│       │                                        │
│       │ (HTTPS, outbound only)                 │
│       ▼                                        │
│  Anthropic API ◄─── message relay              │
│                                                │
└────────────────────────────────────────────────┘
         ▲               ▲               ▲
         │               │               │
    ┌────┴────┐    ┌─────┴─────┐   ┌─────┴─────┐
    │ Terminal │    │  Browser  │   │  Phone    │
    │ (desk)   │    │ (other PC)│   │ (couch)   │
    └─────────┘    └───────────┘   └───────────┘
```

**Security notes**:
- Your computer does **not open any inbound ports** (no direct external access)
- All communication goes through **HTTPS outbound** via the Anthropic API
- Same security level as a regular Claude Code session

## Remote Control vs. Claude Code on the Web

Both use the claude.ai/code interface, but **the execution location is different**.

| | Remote Control | Claude Code on the Web |
|---|---|---|
| **Runs on** | Your computer | Anthropic's cloud |
| **Local file access** | Yes | No |
| **MCP servers** | Your config, as-is | Not available |
| **Best for** | Continuing in-progress local work | Starting fresh without cloning a repo |
| **Parallel tasks** | 1 remote session at a time | Multiple tasks in parallel |

> **Quick rule**: "Already working on something" → Remote Control / "Starting fresh" → Web

## How to Use

### 1. Start a new session

```bash
claude remote-control
```

A **session URL** and **QR code** appear in the terminal. Press spacebar to toggle the QR code.

### 2. Switch from an active session

While already using Claude Code:

```
/remote-control
```

Or shortened:

```
/rc
```

Remote access is enabled while keeping your current conversation history intact.

### 3. Connect from another device

Three options:

| Method | Description |
|---|---|
| **Direct URL** | Open the session URL shown in the terminal in a browser |
| **QR code scan** | Scan with your phone camera → opens directly in the Claude app |
| **Find in app** | Check session list on claude.ai/code or the mobile app (green dot = online) |

### 4. Always-on Remote Control

To skip the command and have it activate automatically for every session:

```
/config
→ "Enable Remote Control for all sessions" → true
```

## Limitations

| Limitation | Details |
|---|---|
| Sessions | Only 1 remote session at a time |
| Terminal | Closing the terminal ends the session |
| Network | Session times out after ~10 minutes without network |
| Auto-reconnect | Laptop sleep → auto-reconnects on wake |

## Real-World Scenarios

### 1. Desk → Couch
```
Fixing a bug at your desk with Claude Code
→ Type /rc → scan the QR code
→ On the couch, tell your phone "fix this file too"
→ Executes on your computer, results visible on your phone
```

### 2. Monitoring a Long-Running Task
```
Start a large refactoring job
→ Launch with claude remote-control
→ Check progress on your phone while out
→ Give additional instructions from your phone if needed
```

### 3. Quick Check During a Meeting
```
Someone asks a code question during a meeting
→ Connect to the running session from your phone
→ Ask "explain the structure of this function"
→ Reads your local code and responds
```

## Requirements

| Item | Requirement |
|---|---|
| Plan | Max (Pro support coming soon) |
| Authentication | `/login` with claude.ai completed |
| Client | Claude Code CLI in a terminal |
| Connecting device | claude.ai/code or iOS/Android Claude app |

## Key Takeaways

- Remote Control = a **remote** for your local Claude Code session (access from phone or browser)
- Execution always stays **on your machine** → local files, MCP servers, and Plugins all available
- Activate with `claude remote-control` or `/rc` → connect via URL or QR code
- Don't confuse with Claude Code on the Web: Remote Control = **local execution**, Web = **cloud execution**
- Terminal must stay open; session times out after ~10 minutes of network loss
