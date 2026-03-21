# Subagent — A Worker That Operates in Its Own Space

## What Is a Subagent?

When you chat with Claude Code, there is **one main agent** in the conversation. Handling large tasks directly in the main conversation, however, quickly fills the context window (200k tokens).

Subagents solve this problem: they **work in an isolated space and report only the result back to the main agent**.

## Analogy from No-Code Tools

| No-code concept | Claude Code Subagent |
|---|---|
| Zapier's Sub-Zap (sub-workflow) | Branches from the main flow, performs a separate task, returns the result |
| Make's Sub-scenario | Processes complex logic in isolation |
| Notion's linked database | Source data lives elsewhere; only the needed information is surfaced |

## Why Is It Needed?

A concrete scenario:

**Without Subagent** — asking to analyze a 200-page PDF:
```
Main context: [existing conversation] + [all 200 pages of PDF] → token explosion
Other tasks are blocked because the context is already full
```

**With Subagent** — the same request:
```
Main context: [existing conversation] + [3-line summary received] → context stays clean
Other tasks can continue
```

## How It Works: Blank Slate

The defining characteristic of a Subagent is that it starts from a **blank slate**.

```
┌─ Main Claude ──────────────────────────┐
│  Conversation context: [A, B, C, D...] │
│                                        │
│  "Analyze this PDF" ──┐                │
│                        ▼               │
│  ┌─ Subagent ──────────────────┐       │
│  │ Context: [blank slate]      │  ← Starts with nothing
│  │ Task: Analyze PDF           │       │
│  │ (reads 200 pages...)        │       │
│  │ Result: 3-line summary ─────┼──▶ returned
│  └─────────────────────────────┘       │
│                                        │
│  Main receives only the 3-line summary │
│  → context preserved                   │
└────────────────────────────────────────┘
```

**What "blank slate" means:**
- Does **not** inherit the main conversation's history
- The Subagent has no knowledge of what has been discussed
- It starts with only the **information relevant to its assigned task**
- This is a feature, not a bug: focused work without irrelevant context

## Subagent Characteristics

| Characteristic | Description |
|---|---|
| **Blank Slate** | Starts with no history from the main conversation |
| **Isolated context** | Even if the Subagent reads 200 files, the main context stays clean |
| **Result-only return** | The working process is discarded; only a summarized result is reported back |
| **1:1 relationship** | Reports only to the main agent; cannot communicate directly with other Subagents |
| **One-time use** | Disappears after completing its task (not persistent) |

## Built-in Subagent Types

Claude Code provides built-in Subagents optimized for different purposes.

| Subagent | Available tools | Model | Purpose | Analogy |
|----------|----------------|-------|---------|---------|
| **Explore** | Read-only (Read, Glob, Grep) | Haiku (fast and cheap) | File exploration, codebase analysis | Librarian |
| **Plan** | Read-only | Sonnet/Opus | Implementation planning, architecture design | Architect |
| **General-purpose** | All tools | Sonnet/Opus | Complex multi-step tasks | All-purpose assistant |
| **Bash** | Bash only | Sonnet/Opus | Running terminal commands | System administrator |

### When to Use Each Subagent

**Explore** — "What's the structure of this project?"
- Quickly scan a codebase
- Uses the cheaper Haiku model
- Safe because it does not modify files

**Plan** — "How should I implement this feature?"
- Reads code and produces a plan only
- Does not write any actual code

**General-purpose** — "Analyze this PDF and summarize it"
- Can use all tools, so handles complex tasks
- Can read/write files and run terminal commands

**Bash** — "Run the full test suite and report back"
- Executes only terminal commands
- Reports only the outcome of long builds or test runs

## Creating Custom Subagents

When the built-in Subagents are not enough, you can define your own.

### File Location
```
project/
└── .claude/
    └── agents/
        └── my-reviewer.md
```

### Example

```markdown
---
name: code-reviewer
description: Reviews code for quality and security issues
tools: Read, Grep, Glob
model: sonnet
---

You are a senior code reviewer.

## Review Checklist
1. Security vulnerabilities (SQL injection, XSS, etc.)
2. Performance issues (unnecessary loops, memory leaks)
3. Code readability
4. Missing error handling

## Output Format
| File | Line | Severity | Issue |
|------|------|----------|-------|
```

### Advantages of Custom Subagents

- **Tool restrictions**: use the `tools` field to limit available tools → enables safe automation
  - Example: a read-only DB query agent (only Read and Grep allowed)
- **Model selection**: cheap Haiku for simple tasks, Opus for complex ones
- **Fixed role**: consistent behavior across all executions

## When Subagents Are Triggered Automatically

Even without explicit invocation, Claude Code may spin up Subagents on its own:

- When codebase exploration is needed → **Explore** Subagent launched automatically
- When a complex implementation plan is needed → **Plan** Subagent launched automatically
- When parallel processing is more efficient → multiple Subagents launched simultaneously

## Key Takeaways

- Subagent = a **sub-agent that works in isolation and reports only the result**
- Starts from a blank slate → prevents contamination of the main context
- 4 built-in Subagents (Explore, Plan, General-purpose, Bash) plus custom support
- Custom Subagents are defined as markdown files in `.claude/agents/`
- 1:1 relationship with the main agent — Subagents cannot communicate directly with each other (this is what distinguishes them from Agent Teams)
