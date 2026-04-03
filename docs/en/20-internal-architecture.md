# Claude Code Internal Architecture — What the Source Code Reveals

## How Is Claude Code Built?

On March 31, 2026, Claude Code's source code was made public. **512,000 lines of TypeScript**, 1,884 files. Analyzing this code reveals exactly how the tool we use every day works under the hood.

Claude Code is built on three design principles:

| Principle | Meaning | Implementation Example |
|---|---|---|
| **Safety** | Protect the user's system | Permission Pipeline, pre-execution authorization checks |
| **Performance** | Fast responses, efficient resource use | Lazy Import, Memoized Context, parallel tool execution |
| **Extensibility** | Easy to add new features | MCP protocol, Hook system, Plugin architecture |

The tech stack consists of **TypeScript + React Ink** (terminal UI rendering) + **Zustand** (state management) + **Bun** (runtime).

## Comparison with No-Code Tools

### 4-Phase Execution = Zapier's Trigger-Filter-Action-Output

Claude Code internally operates through 4 phases. This maps directly to Zapier's workflow structure:

```
Zapier:
  Trigger → Filter → Action → Output

Claude Code:
  Startup → Query Loop → Tool Execution → Display
```

| Zapier | Claude Code | Description |
|---|---|---|
| Trigger (event detection) | Startup (initialization) | Environment loading, CLAUDE.md parsing, permission checks |
| Filter (condition check) | Query Loop (query analysis) | Interpret user input, decide which tools to use |
| Action (execution) | Tool Execution | Read files, write code, run commands |
| Output (result) | Display | Render results to terminal via React Ink |

### 7 Execution Modes = Notion AI's Multiple Views

Just as Notion lets you view the same database as a table, board, gallery, or calendar, Claude Code can run the same core engine in **7 different modes**:

```
Same engine, different interfaces:
  Notion: Table View / Board View / Gallery View / Calendar View
  Claude Code: Interactive / Non-interactive / Pipe / Print / API / MCP / Tool
```

### Context Injection = Cursor's .cursorrules Loading

Just as Cursor automatically reads `.cursorrules` when opening a project to customize AI behavior, Claude Code also automatically injects **CLAUDE.md, Memory, and environment info** into its context at startup.

```
Cursor startup:
  Read .cursorrules → Apply rules to AI

Claude Code startup:
  Load CLAUDE.md → Load Memory → Detect environment → Build context
```

## How It Works

### Startup: 6-Step Initialization

When you launch Claude Code, 6 steps execute sequentially before the prompt appears:

```
1. Environment detection (OS, Shell, Git status)
2. Settings file loading (settings.json)
3. CLAUDE.md parsing (project + global)
4. Auto Memory loading
5. MCP server connections
6. Permission configuration check
```

### 7 Execution Modes

| Mode | Description | Usage Example |
|---|---|---|
| **Interactive** | Conversational terminal use | Run `claude` and chat directly |
| **Non-interactive** | Execute one command and exit | `claude -p "Analyze this file"` |
| **Pipe** | Pass data via stdin/stdout | `cat file.ts \| claude -p "Review this"` |
| **Print** | Output system prompt only | For debugging/verification |
| **API** | External HTTP invocation | Use Claude Code from other programs |
| **MCP** | Operate as an MCP server | Other AI agents use Claude Code as a tool |
| **Tool** | Operate as an SDK tool | Programmatic integration |

### 4-Phase Execution Model in Detail

```
Phase 1: Startup
  ┌─────────────────────────────────────┐
  │ Env detection → Settings → CLAUDE.md│
  │ → Memory → MCP → Permissions       │
  └─────────────┬───────────────────────┘
                │
Phase 2: Query Loop
  ┌─────────────▼───────────────────────┐
  │ Receive user input                  │
  │ → Combine system prompt + context   │
  │ → API call (SSE streaming)          │
  └─────────────┬───────────────────────┘
                │
Phase 3: Tool Execution
  ┌─────────────▼───────────────────────┐
  │ Does the response contain tool      │
  │ calls?                              │
  │ ├─ Yes → Check permissions → Run    │
  │ │        → Add result to context    │
  │ │        → Return to Phase 2        │
  │ └─ No  → Proceed to Phase 4        │
  └─────────────┬───────────────────────┘
                │
Phase 4: Display
  ┌─────────────▼───────────────────────┐
  │ Render terminal UI via React Ink    │
  │ → Markdown formatting               │
  │ → Await next input                  │
  └─────────────────────────────────────┘
```

### Agent Loop: 11 Steps

The repetition of Phases 2-3 is the **Agent Loop**. In detail, it breaks down into 11 steps:

| Step | Processing |
|---|---|
| 1 | Receive user message |
| 2 | Assemble system prompt (CLAUDE.md + Memory + environment) |
| 3 | Send API request (SSE streaming) |
| 4 | Begin receiving response tokens |
| 5 | Detect tool calls |
| 6 | Check permissions (Permission Pipeline) |
| 7 | Execute Hooks (PreToolUse) |
| 8 | Execute tool |
| 9 | Execute Hooks (PostToolUse) |
| 10 | Add result to context |
| 11 | Determine if more tool calls needed → if yes, go to step 3; if no, complete response |

### Source Directory Structure

```
claude-code/src/
├── utils/          564 files — Utility functions (largest directory)
├── components/     389 files — React Ink UI components
├── tools/          287 files — Tool definitions (Read, Write, Bash, etc.)
├── hooks/          156 files — Hook system
├── mcp/            134 files — MCP client/server
├── permissions/     98 files — Permission management
├── memory/          87 files — Memory system
└── agents/          69 files — Subagent, Agent Loop
```

`utils/` is the largest directory at 564 files because core infrastructure — context management, token calculation, file system operations, and caching — is concentrated there.

## Practical Scenarios

### "What Happens Internally When You Type a Message"

Here's the complete process when you type `Analyze the login flow in src/auth/`:

```
1. [Query Loop] Message received
   → "Analyze the login flow in src/auth/"

2. [Query Loop] System prompt assembled
   → CLAUDE.md rules + Auto Memory + current Git status + OS info

3. [Query Loop] API request (SSE streaming)
   → Full context sent to Claude

4. [Tool Execution] Claude decides to call a tool
   → "I need to check the src/auth/ directory first" → Glob tool call

5. [Permission] Authorization check
   → Glob is read-only → auto-approved

6. [Hook] PreToolUse check
   → No registered Hooks → pass

7. [Tool Execution] Glob executes
   → Found src/auth/login.ts, src/auth/session.ts, src/auth/middleware.ts

8. [Query Loop] Result added to context → another API call
   → "Now I need to read these files" → Read tool called 3 times

9. [Tool Execution] Read executed 3 times in parallel (safe tool)
   → All 3 files read simultaneously

10. [Query Loop] File contents added to context → another API call
    → Claude generates the analysis

11. [Display] Markdown formatted via React Ink and rendered to terminal
    → Analysis results shown to user
```

In this process, the Agent Loop iterated **3 times** (Glob once + Read 3x parallel + final response). This is why Claude Code is an **agent**, not a chatbot — it judges on its own, selects tools, and decides next actions based on results.

## Key Takeaways

- Claude Code = an agent system of **512K lines of TypeScript**, 1,884 files
- Tech stack: TypeScript + React Ink (TUI) + Zustand (state) + Bun (runtime)
- **3 design principles**: Safety, Performance, Extensibility
- **4-Phase execution model**: Startup → Query Loop → Tool Execution → Display
- **7 modes**: Interactive, Non-interactive, Pipe, Print, API, MCP, Tool
- **The 11-step Agent Loop** is the core — it repeats until no more tool calls are needed
- Source structure: utils (564) > components (389) > tools (287) are the largest directories
