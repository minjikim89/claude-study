# Context Window Management in Practice — The Reality of 200k Tokens

## What Is a Context Window?

The **context window** is the total amount of text Claude can "see" at one time. Claude Code currently supports approximately **200k tokens** (roughly 150,000 English words).

That sounds like a lot — but in practice, it fills up faster than you'd expect.

## Comparison with No-Code Tools

| No-Code Tools | Claude Code |
|---|---|
| Execution history stored separately | All conversation accumulates inside the context window |
| Previous results remain accessible | Conversation is lost when the session ends |
| Data volume limits, but no conversation length limit | **Performance degrades as conversations grow longer** |

This is exactly why Claude Code's memory systems exist — CLAUDE.md, Memory, Skills, and Subagents. They are all **designed to use a finite context window efficiently**.

## How Much Is 200k Tokens, Really?

### Token Consumption Reference

| Item | Approximate Token Count |
|---|---|
| 1 English word | ~1 token |
| 1 line of code | ~10–20 tokens |
| 1 typical chat message | ~50–200 tokens |
| 1 file with 500 lines of code | ~5,000–10,000 tokens |
| 1 long Claude response | ~500–3,000 tokens |
| CLAUDE.md (concise) | ~200–500 tokens |

### When Context Fills Up Fast

```
Risky scenario:
  "Read all the code in this project"
  → 20 files × avg 5,000 tokens = 100,000 tokens (half gone)
  → Only half the window left for the rest of the conversation

Safe scenario:
  "Read only the login-related files in src/auth/"
  → 3 files × 5,000 tokens = 15,000 tokens (7.5% used)
  → Plenty of room to continue
```

## How Context Gets Consumed

When a session starts, the context fills up in the following order:

```
200k tokens total
┌──────────────────────────────────────────┐
│ ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ↑                                        │
│ System prompt                            │
│ (CLAUDE.md + Auto Memory)                │
│ ~1,000–3,000 tokens                      │
│                                          │
│ Remaining: ~197,000 tokens               │
│ → Used for conversation + file reads     │
│   + Skill loading                        │
└──────────────────────────────────────────┘

After conversation progresses:
┌──────────────────────────────────────────┐
│ ████████████████████████████░░░░░░░░░░░░ │
│ ↑ System   ↑ Conversation history  ↑ Free│
│                                          │
│ The longer the conversation,             │
│ the less free space remains              │
└──────────────────────────────────────────┘
```

## What Happens When Context Runs Low

When Claude Code approaches the context limit, it performs **automatic compression**:

```
Context limit reached
  │
  ▼
Previous conversation auto-summarized/compressed
  │
  ▼
Summary replaces original → space reclaimed
  │
  ▼
Conversation can continue (but detail may be lost)
```

**The problem**: Compression can lose details from earlier in the conversation. Claude may not be able to accurately answer questions like "What was the variable name on line 27 of that third file?"

## Token-Saving Strategies

### 1. Narrow the Scope of Your Requests

```
❌ "Analyze the entire structure of this project"
   → Tries to read all files → token explosion

✅ "Analyze the login flow in src/auth/"
   → Reads only relevant files → tokens saved
```

### 2. Delegate Heavy Work to Subagents

```
❌ In the main conversation: "Review all 100 files"
   → Main context gets packed

✅ "Review all 100 files and only tell me about the problematic ones"
   → Subagent handles it in an independent space
   → Only the summary is returned to the main context
```

### 3. Start a New Session

```
Situation: 50+ turns in, Claude's responses are slow or inaccurate

Steps:
  1. Ask Claude to summarize conclusions and decisions so far
  2. Write important content to CLAUDE.md or a file
  3. Start a new session (Ctrl+C → re-run claude)
  4. Continue with a clean context
```

### 4. Keep CLAUDE.md Concise

```
❌ Copying the entire project documentation into CLAUDE.md
   → Thousands of tokens consumed every session

✅ Only core rules in CLAUDE.md; detailed content split into Skills
   → Loaded on demand → tokens saved
```

### 5. Use the `/clear` Command

```
When the topic shifts completely mid-conversation:
  /clear → resets conversation history (CLAUDE.md and Memory are preserved)
  → Start the new topic with a clean slate
```

## Token Impact by Feature

| Feature | Token Impact | Management Strategy |
|---|---|---|
| **CLAUDE.md** | Always loaded every session | Keep short (under 500 tokens recommended) |
| **Auto Memory** | Always loaded every session | Auto-capped at ~200 lines |
| **Skills** | Loaded only when invoked | Long Skills are fine (on-demand) |
| **File reads** | Proportional to content read | Only read the files you need |
| **Subagents** | Only result returned to main | Use aggressively for bulk processing |
| **Conversation history** | Accumulates continuously | Periodically use /clear or start a new session |

## Warning Signs and Responses

| Sign | What It Means | Response |
|---|---|---|
| Responses getting slower | Context is full, more processing overhead | Start a new session |
| Claude misremembers earlier conversation | Auto-compression lost detail | Record key content to a file, then start new session |
| "Context limit" type message | Limit reached | Switch to a new session immediately |
| Keeps trying to re-read the same file | Previously read content lost to compression | Consider delegating to a Subagent |

## Key Takeaways

- Context window of 200k tokens = the total Claude can see at one time
- Everything (system prompt + conversation + file reads) shares this space
- Long conversations trigger auto-compression → detail can be lost
- **Saving strategies**: narrow scope, delegate to Subagents, start new sessions, keep CLAUDE.md concise
- Claude Code's memory systems (CLAUDE.md, Memory, Skills, Subagents) are all designed to work around this constraint
