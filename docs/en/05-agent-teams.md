# Agent Teams — A Project Team Where Members Talk Directly to Each Other

## What Are Agent Teams?

If Subagents follow a **"subordinate reports only to the main agent"** model, Agent Teams follow a **"team members communicate directly and collaborate"** model.

Multiple Claude instances work simultaneously in independent sessions, coordinating progress through a shared task list.

> Agent Teams are currently an **experimental feature** and must be enabled via a separate configuration.

## The Critical Difference Between Subagents and Agent Teams

```
Subagent structure:
┌─ Main ─────────────────┐
│                        │
│  Subagent A ──▶ Main   │   A reports only to Main
│  Subagent B ──▶ Main   │   B reports only to Main
│  Subagent C ──▶ Main   │   A, B, C cannot communicate with each other
│                        │
└────────────────────────┘

Agent Teams structure:
┌─ Leader ───────────────────────────────────┐
│                                            │
│  ┌─ Agent A ─┐  ┌─ Agent B ─┐  ┌─ Agent C ─┐
│  │ Research  │  │ Report    │  │ Slides    │
│  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
│        │              │              │
│        └──────── messages ───────────┘
│                    +
│            shared task list
└────────────────────────────────────────────┘
```

## Detailed Comparison

| | **Subagent** | **Agent Teams** |
|---|---|---|
| **Analogy** | Subordinate (1:1 reporting) | Project team (horizontal communication) |
| **Communication** | Returns results to main only | Members message each other + shared task list |
| **Coordination** | Main manages and orchestrates everything | Self-coordinating via task list |
| **Context** | Isolated (result only returned) | Each member has a fully independent session |
| **Parallelism** | Limited (main manages sequentially) | High (simultaneous execution) |
| **Token cost** | Low (one at a time) | High (N sessions simultaneously) |
| **Activation** | Available by default | Requires configuration (experimental) |
| **Best for** | Delegating a single task | Simultaneous collaboration on multiple tasks |

## Comparison with No-Code Tools

| No-code concept | Claude Code |
|---|---|
| Calling an HTTP module from a single Make scenario | Subagent |
| Multiple Make scenarios connected via Webhook | Agent Teams |
| One person working on multiple Notion pages | Subagent |
| Multiple people simultaneously writing their own Notion pages | Agent Teams |
| Manager giving 1:1 task instructions in Slack | Subagent |
| Team members collaborating directly in a Slack channel | Agent Teams |

## How to Enable Agent Teams

Add the following to `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Restart Claude Code after saving.

## How Agent Teams Work

### 1. Leader-Member Structure

```
User: "Work on the frontend, backend, and tests simultaneously"
  │
  ▼
Leader Claude (main session)
  │
  ├─▶ Agent A: Frontend development (independent session)
  ├─▶ Agent B: Backend API development (independent session)
  └─▶ Agent C: Test code authoring (independent session)
```

### 2. Shared Task List

Team members track each other's progress through a **shared task list**:

```
[Shared Task List]
├── ✅ Agent A: Component structure design — complete
├── 🔄 Agent B: API endpoint implementation — in progress
├── ⏳ Agent C: Waiting for A and B (dependency)
└── 📋 Unassigned: Write integration tests
```

### 3. Direct Messages Between Members

Unlike Subagents, Agent Teams members can **message each other directly**:

```
Agent A (frontend): "Is the API response shape { data: [] }?"
Agent B (backend): "Yes, and errors return { error: message }"
Agent C (tests): "I'll cover both shapes in the test cases"
```

## Choosing Between Subagent and Agent Teams

### Use Subagent When

- **Delegating a single task**: "Summarize this PDF", "Explore the codebase structure"
- **Sequential pipeline**: A completes → B starts → C starts
- **Cost-conscious**: minimizing token usage is a priority
- **Simple analysis/reporting**: only the result matters, not the process

### Use Agent Teams When

- **Parallel development**: "Develop frontend, backend, and tests simultaneously"
- **Multi-perspective review**: "Three specialists review the PR at the same time"
- **Interdependent tasks**: A's output feeds B, and B's output feeds C
- **Large-scale refactoring**: multiple modules need to be modified at once

## Practical Examples

### Example 1: PR Review (Agent Teams)

```
Leader: "Review this PR from three angles"
  │
  ├─▶ Security specialist: checks for SQL injection, XSS, etc.
  ├─▶ Performance specialist: checks for N+1 queries, memory leaks, etc.
  └─▶ Architecture specialist: checks pattern consistency, coupling, etc.
  │
  ▼
Leader: aggregates three reviews into final feedback
```

### Example 2: Full-Stack Feature Development (Agent Teams)

```
Leader: "Add a user profile edit feature"
  │
  ├─▶ Agent A: React component + form UI
  ├─▶ Agent B: Express API endpoint
  └─▶ Agent C: Jest + Cypress tests
  │
  ▼
[A to B]: "We'll also need a profile image upload API"
[B to C]: "Upload API endpoint is at /api/profile/image"
[C]: "I'll add upload API tests as well"
```

## Costs and Trade-offs

| | Subagent | Agent Teams |
|---|---|---|
| **Token usage** | Low (one at a time) | High (N sessions simultaneously) |
| **Time** | Can be slow (sequential) | Fast (parallel) |
| **Complexity** | Simple | Coordination overhead exists |
| **Output quality** | Single perspective | Multiple perspectives combined |

> **General rule**: If tasks are independent and simple, use Subagents. If tasks are interdependent and complex, use Agent Teams.

## Key Takeaways

- Agent Teams = **multiple Claude instances working simultaneously in independent sessions with direct communication**
- Key difference from Subagents: **direct messages** between members + **shared task list**
- Currently experimental → must be enabled in `settings.json`
- Higher token cost, but gains in speed and quality through parallelism
- In most real-world work, Subagents are sufficient; Agent Teams become relevant for large, complex projects
