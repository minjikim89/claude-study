# Agent Loop Anatomy — The Core That Makes an Agent

## What Is the Agent Loop?

The **Agent Loop** is exactly what makes Claude Code an "agent" rather than a "chatbot." A regular chatbot receives a question, generates an answer, and stops. Claude Code, on the other hand, receives a question and then **autonomously decides which tools to call, checks the results, and repeats the process until the task is complete**.

At the heart of this loop is the `query()` async generator function. It receives responses in real time via SSE (Server-Sent Events) streaming while detecting and executing tool calls.

```
Chatbot:
  Question → Answer (1 round)

Agent:
  Question → Tool call → Check result → More tool calls → ... → Final answer (N rounds)
```

## Comparison with No-Code Tools

### Agent Loop = Make.com's Scenario Execution

Just as a Make.com scenario can branch to different paths via a Router based on conditions, the Agent Loop branches at every iteration — the AI decides whether to use a tool or not.

```
Make.com:
  Trigger → Module 1 → Router → [Condition A] Module 2 → Module 3
                               → [Condition B] Module 4 → End

Agent Loop:
  User query → API call → [Tool needed] → Execute → Result → API call → ...
                         → [No tool needed] → Final response
```

### Tool Detection = Zapier's "If This Then That" (But AI Decides the "If")

In Zapier, a human sets the conditions: "When an email arrives in Gmail, send a Slack notification." In the Agent Loop, **the AI decides the condition itself**: "I need to know the file structure, so I'll use Glob."

### Auto-Compact = Gmail's Auto-Archive

Just as Gmail automatically archives old emails when your inbox is full, Claude Code **automatically summarizes and compresses** earlier conversations when the context window fills up.

### SSE Streaming = Real-Time Editing in Google Docs

Just as you can see someone else typing in real time in Google Docs, Claude Code streams API responses token by token in real time. Instead of waiting for the full answer to complete, it appears on screen as it's being generated.

## How It Works

### 11-Step Agent Loop in Detail

| Step | Processing | Details |
|---|---|---|
| 1 | Message received | User input or tool execution result |
| 2 | Context assembly | System prompt + conversation history + tool results |
| 3 | API request | Sent via SSE streaming |
| 4 | Token reception | Real-time token-by-token reception begins |
| 5 | Tool call detection | Detect tool_use blocks in the response stream |
| 6 | Permission check | Permission Pipeline determines if execution is allowed |
| 7 | PreToolUse Hook | Execute registered Hooks if any |
| 8 | Tool execution | Actual file read/write/command execution |
| 9 | PostToolUse Hook | Execute registered Hooks if any |
| 10 | Result added | Tool execution result added to context |
| 11 | Loop decision | More tools needed → go to step 3 / Done → output response |

### Streaming Tool Executor: Overlapped Execution

A typical implementation would "receive full response → then execute tools," but Claude Code **starts tool execution while still receiving the response**. This is called the Streaming Tool Executor.

```
Typical implementation:
  [===Receiving===] → [Tool exec] → [===Receiving===] → [Tool exec]

Claude Code:
  [===Receiving===[Tool exec starts]===] → [===Receiving===[Tool exec]===]
                   ↑ Overlap
```

This overlap makes the perceived response time significantly faster.

### Tool Concurrency Model: Safe Tools Run in Parallel, Unsafe Run Sequentially

Not all tools execute the same way:

| Category | Execution | Max Concurrent | Tools |
|---|---|---|---|
| **Safe (read-only)** | Parallel | Up to 10 | Read, Glob, Grep, ListMcpResources |
| **Unsafe (modifying)** | Sequential | 1 at a time | Write, Bash, Edit |

```
Reading 10 files simultaneously:
  Read(file1) ─┐
  Read(file2) ─┤
  Read(file3) ─┤
  ...          ├─ Concurrent execution → All results returned
  Read(file9) ─┤
  Read(file10)─┘

Writing 3 files sequentially:
  Write(file1) → Done → Write(file2) → Done → Write(file3)
```

The rationale: reads don't affect each other, but writes require ordering (writing to the same file simultaneously would cause conflicts).

### Auto-Compact: Automatic Handling When Context Fills Up

When the context window fills up, Claude Code automatically performs compression. This process is not simple:

```
Auto-Compact flow:

  Check context usage
  │
  ├─ Below threshold → proceed normally
  │
  └─ Above threshold (context_window - 13,000 tokens)
     │
     ▼
     Spawn sub-agent (dedicated summarizer)
     │
     ├─ Success → Replace earlier conversation with summary
     │            → Restore file contents up to 50K tokens
     │            → Re-inject Skill content up to 25K tokens
     │            → Continue conversation
     │
     └─ Failure → Increment circuit breaker count
                  → After 3 consecutive failures, stop compressing
                  → Recommend user start a new session
```

Key numbers:

| Item | Value |
|---|---|
| Compression trigger threshold | context_window - 13,000 tokens |
| Circuit breaker limit | 3 consecutive failures |
| File content restoration limit | Up to 50K tokens |
| Skill re-injection limit | Up to 25K tokens |

### The 3-Line Fix That Saved 250K Tokens/Day

An interesting optimization found in the source code: unnecessary context was being repeatedly sent with every API request. Adding **3 lines of cache key logic** eliminated approximately 250K tokens per day of wasted API calls.

```
Problem:
  System prompt sent in full with every API request
  → Same content billed as tokens each time

Fix (3 lines):
  Set cache breakpoints on the system prompt
  → API reuses cache from previous requests
  → Eliminates redundant transmission within the same session
```

This is the practical value of analyzing open-source code — you can learn about real performance problems and their solutions in production.

## Practical Scenarios

### "Your Context Window Is Filling Up — What Happens Next?"

The Auto-Compact process when a conversation gets long:

```
Conversation state:
  ████████████████████████████████░░░░ (~90% used)

1. Agent Loop prepares the next API request
2. Context size check → threshold exceeded detected
3. Auto-Compact triggered

4. Sub-agent spawned (dedicated summarizer)
   → "Summarize the previous conversation, keeping only key points"
   → Sub-agent generates summary in an independent context

5. Replace with summary
   Before: [msg1][msg2]...[msg50] = 150K tokens
   After:  [Summary: 5 key decisions] = 5K tokens

6. Restore important files
   → Reload recently read files up to 50K
   → Reduce the rest to "read file1.ts previously" level

7. Re-inject Skills
   → If active Skills existed, re-inject up to 25K

8. Conversation continues
   ████████░░░░░░░░░░░░░░░░░░░░░░░░░░ (~30% used)
```

### "Claude Reads 10 Files at Once" — Parallel Batching Explained

When you request a large-scale code analysis:

```
User: "Read all TypeScript files in src/ and analyze the architecture"

1. Glob executes → 12 .ts files found

2. Claude decides to call Read 12 times

3. Tool Executor classifies:
   Read = Safe (read-only) → parallel execution allowed

4. Batched execution:
   Batch 1: Read(file1~file10)  → 10 executed simultaneously (max concurrent)
   Batch 2: Read(file11~file12) → 2 executed simultaneously

5. Result: all 12 files read in 2 rounds
   Sequential would have required: 12 rounds
   Parallel result: reduced to 2 rounds (6x faster)
```

You don't need to worry about this parallel execution — Claude Code automatically determines tool safety and selects the optimal execution strategy.

## Key Takeaways

- Agent Loop = the core repetition structure that makes Claude Code an **agent**
- `query()` async generator handles SSE streaming + tool detection + execution in one flow
- **Streaming Tool Executor**: overlaps tool execution with response reception for faster perceived speed
- **Tool concurrency**: Safe tools run up to 10 in parallel, Unsafe tools run sequentially
- **Auto-Compact**: sub-agent summarization on threshold breach, 50K file restore, 25K Skill re-inject, circuit breaker after 3 failures
- 3-line cache optimization saved 250K/day in wasted API calls — real-world optimization from production code
