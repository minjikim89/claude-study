# Claude Code System — Memory Architecture and MD File Levels

## How It Works

At first glance, Claude Code feels like an AI chatbot. In reality, it is an **AI agent that operates on top of the file system**. The core mechanism is simple: **injecting text into the context window (200k tokens)**.

Every configuration in Claude Code — rules, memory, Skills — is ultimately a markdown file. These files are loaded into the context window at the right moment.

## Chat Interface vs. CLI — A Fundamental Difference

If you come to Claude Code from experience with claude.ai (web/app), the two feel similar but work in fundamentally different ways.

### Chat Interface (claude.ai web/app)

- Type messages in a **browser or mobile app**
- Conversations are stored on **Anthropic's cloud servers**
- Conversations started on the web can be resumed on mobile (synced across the same account)
- What the AI can do: text generation, analysis, summarization — **work within the conversation**
- Cannot access files on your computer

### CLI (Claude Code)

- Run via the `claude` command in a **terminal**
- Conversations exist **only on your local machine**
- When the session ends, conversation context is gone (use CLAUDE.md and Memory for persistence)
- What the AI can do: read/write files, run terminal commands, manipulate Git — **operate across your entire computer**
- Works directly inside your project folder

### At a Glance

| | Chat (claude.ai) | CLI (Claude Code) |
|------|-------------------|-------------------|
| Runtime | Browser / mobile app | Terminal (macOS, Linux) |
| Conversation storage | Cloud (account-synced) | Local (per session, not synced) |
| File access | None (uploaded files only) | Full filesystem access |
| Code execution | None | Bash, Python, etc. — directly |
| External integrations | Limited | Slack, Notion, DB, etc. via MCP |
| Analogy | Texting an assistant with questions | Assistant coming to your office to work |

**Key point**: Both use the same Claude model (Opus, Sonnet, etc.), but the **scope of what they can access** is completely different. Chat is confined to "within the conversation"; CLI uses "the entire computer" as its workspace.

## Chat and CLI Are Not Connected

Conversations in the Chat Interface and conversations in the CLI are **completely separate**.

```
claude.ai (web/app)  ←→  Synced  (same account, conversations carry over)
        ↕
     No connection  ← No sharing of history, settings, or context
        ↕
Claude Code (CLI)  →  Local session only; context gone when session ends
```

| | Chat → CLI | CLI → Chat |
|------|-----------|-----------|
| Continuing a conversation | Not possible | Not possible |
| Sharing settings | Not possible (Projects ≠ CLAUDE.md) | Not possible |
| Sharing files | Not possible | Not possible |

### Why Are They Separate?

Claude Code is an **agent that operates on the local filesystem**. Conversations are tightly coupled to local operations — reading/writing files, running terminal commands — which makes it structurally incompatible with sharing conversation history with the cloud-based claude.ai.

### Comparison with No-Code Tools

| No-code tool | Sync model | Claude ecosystem |
|---|---|---|
| Notion web ↔ Notion app | Fully synced | claude.ai web ↔ app: synced |
| Notion ↔ Zapier | Connected via API (separate products) | claude.ai ↔ Claude Code: not connected |
| Cursor ↔ ChatGPT | Completely separate products | Analogous relationship |

**Summary**: Do not think of claude.ai and Claude Code as "two interfaces to the same tool," the way Notion web and mobile are. They are closer to **Notion and Zapier** — separate products that happen to come from the same company. They share the underlying AI model, but the interfaces, storage, and capabilities are entirely different.

## Context Window Structure

```
┌─────────────────────────────────────────────────────────┐
│                  Context Window (200k tokens)            │
│                                                          │
│  ┌──────────────┐                                        │
│  │ CLAUDE.md    │ ← Always auto-loaded at session start  │
│  │ (global rules)│                                       │
│  └──────────────┘                                        │
│                                                          │
│  ┌──────────────┐                                        │
│  │ Auto Memory  │ ← Always auto-loaded (MEMORY.md)       │
│  │ (learned notes)│                                      │
│  └──────────────┘                                        │
│                                                          │
│  ┌──────────────┐                                        │
│  │ Skill A      │ ← Loaded only when "/command" is typed │
│  │ (recipe)     │                                        │
│  └──────────────┘                                        │
│                                                          │
│  ┌──────────────┐                                        │
│  │ Conversation │ ← Messages exchanged with Claude       │
│  └──────────────┘                                        │
└─────────────────────────────────────────────────────────┘
```

200k tokens is not infinite. The design reflects this: **always-needed things are auto-loaded**, **occasionally-needed things are on-demand**, and **ephemeral things live only in the conversation**.

## CLAUDE.md — The Project Rulebook

### Three-Level System

CLAUDE.md exists at three levels depending on scope.

```
~/.claude/CLAUDE.md           ← Global: applies to all projects (personal preferences)
./CLAUDE.md                   ← Project: applies to this project (team-shared)
./CLAUDE.local.md             ← Local: applies to this project for you only (git-ignored)
```

When all three files are present, they are **all merged** and injected into the system prompt.

### Purpose and Analogy per Level

| Level | File path | Scope | Analogy | Example |
|---|---|---|---|---|
| **Global** | `~/.claude/CLAUDE.md` | All my projects | Personal work principles | "Reply in English", "prefer tables" |
| **Project** | `./CLAUDE.md` | This entire project | Team convention doc | "Run tests first", "commit message format" |
| **Local** | `./CLAUDE.local.md` | This project, me only | My desk sticky note | "Always enable debug mode", "ignore this path" |

### Comparison with No-Code Tools

| No-code tool | Claude Code |
|---|---|
| Notion workspace settings | Global CLAUDE.md |
| Per-project settings page | Project CLAUDE.md |
| Personal views / filters | CLAUDE.local.md |

### Writing Tips

- **Keep it short**: loaded every session, so conserve tokens
- **Use imperative voice**: "Respond in English", "Run tests before committing"
- **Structure it**: use headings and lists for readability
- **State priorities**: for rules that might conflict, note which takes precedence

## What Happens at Session Start

When Claude Code launches, the context is assembled in this order:

```
New session starts
  │
  ▼
┌─ Auto-loaded (every time) ──────────────────────────────┐
│                                                          │
│  1. ~/.claude/CLAUDE.md          (global rules)          │
│  2. ./CLAUDE.md                  (project rules)         │
│  3. ./CLAUDE.local.md            (local rules)           │
│  4. Auto Memory (MEMORY.md)      (learned content)       │
│                                                          │
│  → All merged and injected into the system prompt        │
└──────────────────────────────────────────────────────────┘
  │
  ▼
┌─ Standby (loaded on demand) ────────────────────────────┐
│                                                          │
│  5. Skills (SKILL.md files)                              │
│     → Only the list is known; body loaded when invoked   │
│                                                          │
└──────────────────────────────────────────────────────────┘
  │
  ▼
  Conversation begins
```

**No-code analogy**: Like an app loading your settings on launch. Just as Notion applies workspace and personal settings automatically when you open it, Claude Code reads all MD files at the start of every session.

## Full Storage Map

```
~/.claude/
├── CLAUDE.md                          ← Global rules (all projects)
├── settings.json                      ← Claude Code settings
└── projects/
    └── <project-name>/
        └── memory/
            └── MEMORY.md              ← Auto Memory (per-project, auto-learned)

Project root/
├── CLAUDE.md                          ← Project rules (team-shared, in git)
├── CLAUDE.local.md                    ← Local rules (me only, git-ignored)
└── .claude/
    ├── skills/
    │   └── my-skill/
    │       └── SKILL.md               ← Skill (loaded only when called)
    └── agents/
        └── my-agent.md                ← Custom Subagent definition
```

## Key Takeaways

- **Chat (claude.ai) and CLI (Claude Code) are separate products**: no shared conversation history, settings, or files. Same underlying AI model, completely different interfaces and capabilities
- All Claude Code configuration operates through a **markdown file → context injection** mechanism
- CLAUDE.md has three levels (global / project / local) to control scope
- Distinguish between always-loaded (CLAUDE.md, Memory) and on-demand-loaded (Skills) to use tokens efficiently
- At session start, context is assembled in order: auto-load → standby → conversation
