# 12 Design Patterns — Agent Patterns Extracted from Production Code

## Why Design Patterns?

We extracted **8 core patterns + 4 infrastructure patterns** from the Claude Code source code. These aren't mere coding techniques — they are **essential design strategies** for agent systems to operate reliably.

The critical difference between regular applications and agent systems: agents are **unpredictable in what they'll do next**. Depending on user input, they might read files, execute commands, or do nothing at all. Maintaining stability amid this uncertainty requires robust patterns.

## Comparison with No-Code Tools

Each of the 12 patterns mapped to a familiar tool analogy:

### 8 Core Patterns

| # | Pattern | No-Code Analogy |
|---|---|---|
| 1 | **Generator Streaming** | Watching someone type in real time in Google Docs |
| 2 | **Feature Gate** | Notion's Feature Preview toggles — turn features on and off |
| 3 | **Memoized Context** | Browser caching frequently visited pages |
| 4 | **Withhold & Recover** | Autocorrect fixing typos before you even notice |
| 5 | **Lazy Import** | Apps loading only the current screen, loading the rest when you navigate |
| 6 | **Immutable State** | Google Sheets version history — can't accidentally overwrite |
| 7 | **Interruption Resilience** | Google Docs auto-save — content preserved even if the browser crashes |
| 8 | **Dependency Injection** | USB ports — same port, different devices |

### 4 Infrastructure Patterns

| # | Pattern | No-Code Analogy |
|---|---|---|
| 9 | **Tool Concurrency** | Parallel vs sequential cooking — chopping vegetables simultaneously, stir-frying in order |
| 10 | **Auto-Compact** | Summarizing meeting notes when the notebook is full |
| 11 | **Permission Pipeline** | Building security — checks at lobby, elevator, and office door |
| 12 | **API Retry** | Auto-redial when a phone call drops |

## How It Works

### Pattern 1: Generator Streaming

**What**: Instead of receiving the full API response at once, stream and display it token by token in real time.

```
Traditional approach:
  [====Waiting====] → [Show entire response at once]

Generator Streaming:
  [to][ken][by][to][ken] → [Display on screen in real time]
```

**Claude Code implementation**: The `query()` function is implemented as an async generator, yielding each token. React Ink renders them in real time.

### Pattern 2: Feature Gate

**What**: A toggle system that can enable or disable features at runtime.

```
Feature Gate check:
  Feature X requested
  │
  ├─ Gate ON  → Execute feature
  └─ Gate OFF → Fallback path or disabled
```

**Claude Code implementation**: 7 execution modes, Beta feature toggles, and per-MCP-server tool enable/disable are all Feature Gate patterns.

### Pattern 3: Memoized Context

**What**: Cache computed results to prevent redundant recalculation.

```
First call:
  Parse CLAUDE.md → Generate result → Store in cache → Return

Second call:
  Check cache → Result exists → Return directly (skip parsing)
```

**Claude Code implementation**: System prompts, CLAUDE.md, and environment info are cached within a session. 14 cache vectors exist.

### Pattern 4: Withhold & Recover

**What**: Detect problems proactively and recover before the user notices.

```
Typical error handling:
  Execute → Error → Show error to user → Manual recovery

Withhold & Recover:
  Execute → Error detected → Auto-retry/recover → User never knows
```

**Claude Code implementation**: Auto-retry on tool execution failure, auto-recovery on context corruption, backoff retry on API errors.

### Pattern 5: Lazy Import

**What**: Don't load all modules at startup — load them only when needed.

```
Eager Import (slow startup):
  Start → [Load A][Load B][Load C] → Ready

Lazy Import (fast startup):
  Start → [Load core only] → Ready
  ... later when needed → [Load B]
```

**Claude Code implementation**: MCP server connections, Plugin loading, and heavy utilities all use Lazy Import. Minimizes the 888KB initial bundle size.

### Pattern 6: Immutable State

**What**: Instead of modifying state directly, create new state and replace.

```
Mutable (risky):
  state.messages.push(newMsg)  → Previous state is gone

Immutable (safe):
  newState = {...state, messages: [...state.messages, newMsg]}
  → Previous state is preserved
```

**Claude Code implementation**: Conversation history managed immutably in Zustand store. Previous states can be rolled back to.

### Pattern 7: Interruption Resilience

**What**: Design that prevents data loss even when interrupted mid-operation.

```
Interruption scenario:
  Writing file → Ctrl+C → ?

Fragile implementation: File half-written and corrupted
Resilient implementation: Write to temp file → Replace on completion (atomic write)
```

**Claude Code implementation**: Memory auto-save, session state recovery, interrupt handling during tool execution.

### Pattern 8: Dependency Injection (DI)

**What**: Depend on interfaces rather than concrete implementations, making components swappable.

```
Without DI:
  Agent Loop → Directly calls Anthropic API

With DI:
  Agent Loop → API Interface → [Anthropic API / Mock API / Local Model]
  Same code works in different environments
```

**Claude Code implementation**: API client, file system access, and permission checker all use the DI pattern. Swappable with mocks for testing.

### Pattern 9: Tool Concurrency

**What**: Execute safe tools in parallel, dangerous tools sequentially.

```
Safe tools (Read, Glob): Up to 10 in parallel
Unsafe tools (Write, Bash): 1 at a time, sequentially
```

### Pattern 10: Auto-Compact

**What**: When the context limit is reached, a sub-agent automatically summarizes and compresses.

```
Threshold: context_window - 13,000 tokens
On failure: Circuit breaker stops after 3 attempts
Restoration: 50K file content + 25K Skill re-injection
```

### Pattern 11: Permission Pipeline

**What**: Multiple stages of authorization checks must pass before tool execution.

```
Tool execution requested
  │
  ▼
  Stage 1: Check tool type (Safe/Unsafe)
  │
  ▼
  Stage 2: Check project settings (allowlist/denylist)
  │
  ▼
  Stage 3: Check user settings (auto-approve status)
  │
  ▼
  Stage 4: Prompt user for confirmation (if needed)
  │
  ▼
  Execution approved or blocked
```

**Claude Code implementation**: A 23-step permission check pipeline runs before every tool execution.

### Pattern 12: API Retry

**What**: Retry failed API calls with exponential backoff.

```
Attempt 1: Fail → Wait 1s
Attempt 2: Fail → Wait 2s
Attempt 3: Fail → Wait 4s
Attempt 4: Success → Return result

Max retries: typically 3-5
```

### Complete Pattern Summary Table

| # | Pattern | Category | Claude Code Evidence | Key Numbers |
|---|---|---|---|---|
| 1 | Generator Streaming | Core | query() async generator | Real-time token delivery |
| 2 | Feature Gate | Core | 7 execution modes, Beta toggles | 7 mode switches |
| 3 | Memoized Context | Core | System prompt caching | 14 cache vectors |
| 4 | Withhold & Recover | Core | Auto-retry, error recovery | Invisible to user |
| 5 | Lazy Import | Core | MCP/Plugin deferred loading | 888KB bundle minimized |
| 6 | Immutable State | Core | Zustand state management | Rollback-capable |
| 7 | Interruption Resilience | Core | Atomic write, session recovery | Zero data loss on interrupt |
| 8 | Dependency Injection | Core | API/FS/Permission DI | Mock-testable |
| 9 | Tool Concurrency | Infra | Safe=parallel, Unsafe=sequential | Max 10 parallel |
| 10 | Auto-Compact | Infra | Sub-agent summarization | 250K/day savings |
| 11 | Permission Pipeline | Infra | Multi-stage auth checks | 23-step pipeline |
| 12 | API Retry | Infra | Exponential backoff retry | 3-5 retries |

## Practical Scenarios

### "Which Patterns Matter for YOUR Project?"

Different project types call for different priority patterns:

**Web Apps (Next.js, React, etc.)**

| Priority | Pattern | Reason |
|---|---|---|
| High | Lazy Import | Bundle size directly affects load speed |
| High | Immutable State | Foundational principle of React state management |
| Medium | Feature Gate | Essential for A/B testing and gradual feature rollouts |
| Medium | API Retry | Handling unstable network conditions |

**Automation Scripts (Python, Node.js)**

| Priority | Pattern | Reason |
|---|---|---|
| High | API Retry | Stability for scripts with heavy external API dependency |
| High | Interruption Resilience | Data protection when long-running tasks are interrupted |
| Medium | Withhold & Recover | Auto-recover from errors to enable unattended execution |

**AI Agent Systems**

| Priority | Pattern | Reason |
|---|---|---|
| High | Generator Streaming | Improve perceived response speed |
| High | Tool Concurrency | Maximize agent execution efficiency |
| High | Auto-Compact | Stable maintenance of long conversations |
| High | Permission Pipeline | Control the agent's scope of action |
| Medium | Memoized Context | Cost reduction by eliminating redundant computation |
| Medium | DI | Support swapping different models and tools |

### Pattern Combination Synergies

The 12 patterns are individually useful, but combining them creates synergies:

```
Example: "User requests large-scale code analysis"

1. Generator Streaming → Display response in real time
2. Tool Concurrency → Read 10 files in parallel
3. Memoized Context → Reuse already-parsed CLAUDE.md from cache
4. Permission Pipeline → Check authorization before each file read
5. Auto-Compact → Auto-summarize when context fills up
6. API Retry → Auto-retry if an API error occurs mid-process
```

Six patterns working simultaneously to handle a single request. This is what distinguishes an agent system from a simple API wrapper.

## Key Takeaways

- **8 core + 4 infrastructure = 12 design patterns** extracted from the Claude Code source
- The central challenge of agent systems: maintaining stability amid unpredictable execution flows
- Key numbers: 250K/day savings, 23-step permissions, 14 cache vectors, 888KB bundle, max 10 parallel
- Priority patterns differ by project type (web app / automation / AI agent)
- The 12 patterns combine for synergy — multiple patterns work simultaneously on a single request
