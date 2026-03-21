# Agentic Frameworks vs. Claude Code — A Concept-by-Concept Comparison

## Why Compare Them?

The AI agent ecosystem includes "agentic frameworks" like CrewAI, LangGraph, and AutoGen. These define and orchestrate agents through code.

Claude Code takes a fundamentally different approach: **markdown instead of code**, **context injection instead of a runtime**, **natural language instead of configuration**.

But because they address the same underlying problems, mapping concepts side by side is the fastest way to understand Claude Code's architecture.

## Fundamental Philosophy

| | **Agentic Framework** | **Claude Code** |
|---|---|---|
| **Agent definition** | Create classes/objects in code (Python, etc.) | Write instructions in a markdown file |
| **Execution model** | Framework runtime executes agent code | Claude reads the markdown and acts on it |
| **Tool binding** | Bind tool functions to each agent | Claude uses its built-in tools (Bash, Read, Write, etc.) |
| **Entry barrier** | Requires Python + framework API knowledge | Only needs markdown authoring ability |
| **Analogy** | **Hire** a specialist, hand them tools, give them work | **Hand an existing assistant a recipe card** |

## Concept-by-Concept Mapping

### 1. System Prompt / Global Configuration

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
System Prompt / Global       →  CLAUDE.md
  Role                            "Reply in English"
  Backstory                       "AI Native Camp project"
  Constraints                     "Follow the STOP PROTOCOL"
```

The agent personality, role, and constraints you configure in code in an agentic framework are replaced in Claude Code by a single **CLAUDE.md file**. Scope can be adjusted through three levels (global / project / local).

### 2. Task-Specific Prompts

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Task-specific Prompt         →  Skill (SKILL.md)
  Task Instructions               Recipe body
  Reference Data                  references/ folder
  Scripts                         scripts/ folder
```

The Task object assigned to each agent in a framework maps to a **Skill** in Claude Code. Because a Skill is a markdown file, non-developers can write them.

### 3. Memory System

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Long-term Memory             →  CLAUDE.md + Auto Memory
Working Memory               →  Conversation context
Shared Memory                →  Agent Teams task list
```

The long-term memory implemented via vector databases or external storage in frameworks is replaced in Claude Code by a **file-based memory system**. No separate infrastructure — markdown files are the memory.

### 4. Tool Calling

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Custom Tools (function defs) →  Built-in tools (Bash, Read, Write, Glob, Grep, etc.)
External API integration     →  MCP (Model Context Protocol)
```

In frameworks you write tool functions and bind them to agents. Claude Code **ships with built-in tools** for filesystem operations and terminal execution. For external services, extend with **MCP**.

### 5. Task Delegation / Multi-Agent

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Agent Delegation             →  Subagent (isolated space, result-only return)
Multi-agent Orchestration    →  Agent Teams (independent sessions, direct communication)
Sequential Workflow          →  Sequential Subagent calls
Parallel Workflow            →  Agent Teams parallel execution
```

## Full Architecture at a Glance

```
Claude Code Architecture
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Persistent memory  →  CLAUDE.md + Auto Memory     (always loaded)
Recipes            →  Skill                        (loaded on demand)
External access    →  MCP                          (external service plug)
Task delegation    →  Subagent                     (isolated space, result-only)
Team collaboration →  Agent Teams                  (independent sessions, direct comms)
```

## Mapping for No-Code Tool Users

If you are coming from Zapier, Make, or n8n:

| No-code concept | Claude Code equivalent | Notes |
|---|---|---|
| **Workflow template** | Skill | Templatize repetitive tasks |
| **Global settings / Preferences** | CLAUDE.md | Defaults applied to all tasks |
| **Variables / Data storage** | Auto Memory | Automatically saves learned content |
| **External app integration** | MCP | Connects to external services |
| **Parallel execution (Parallel Path)** | Agent Teams | Run multiple tasks simultaneously |
| **Sub-workflow** | Subagent | Delegate part of a task |

## Key Takeaways

- Agentic frameworks are **code-centric**; Claude Code is **text (markdown)-centric**
- Both solve the same problems, but the entry barrier is completely different
- All Claude Code configuration ultimately comes down to **"injecting text into the context window"**
- The complex configuration of frameworks reduces to a few markdown files in Claude Code
