# Hooks — Event-Driven Automatic Execution

## What Are Hooks?

If Skills are **recipes you invoke manually**, Hooks are **scripts that run automatically when a specific event occurs**.

"Auto-format on file save", "lint check before committing" — these kinds of automations are what Hooks enable.

## Comparison with No-Code Tools

| No-code tool | Hooks equivalent |
|---|---|
| **Zapier's Trigger** | Hook event (what triggers it) |
| **Make's Watch Module** | Detects a specific event and executes automatically |
| **GitHub Actions' `on: push`** | Event-driven automated workflow |
| **Notion's automation rules** | Automatic rules like "when page is created → apply template" |

**Key difference**: no-code tools configure triggers in a GUI; Hooks are configured by **registering shell commands in a config file (`settings.json`)**.

## How It Works

```
Claude Code execution flow:

  User request → Claude runs a tool → [Hook check] → Return result
                                           │
                                           ▼
                                  Event matches?
                                  ├─ Yes → Run shell command → Pass result to Claude
                                  └─ No  → Skip
```

Hooks can intercept **before or after** Claude Code runs a tool:

| Timing | Description | Use case |
|---|---|---|
| **PreToolUse** | **Before** the tool runs | Block execution, conditional allow, warn |
| **PostToolUse** | **After** the tool runs | Formatting, lint, logging, notifications |
| **Notification** | When Claude sends a notification | Desktop alerts, Slack notification integration |
| **Stop** | When Claude finishes a response | Final validation, automatic post-processing |

## How to Configure

Add to `~/.claude/settings.json` or your project's `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'File write detected: $CLAUDE_FILE_PATH'"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### Configuration Fields

| Field | Description | Example |
|---|---|---|
| `matcher` | Which tool to react to | `"Write"`, `"Bash"`, `"Read"` |
| `type` | Hook type | `"command"` (run a shell command) |
| `command` | Shell command to execute | `"npx prettier --write ..."` |

### Available Environment Variables

Inside a Hook command, you can use environment variables provided by Claude Code:

| Variable | Description |
|---|---|
| `$CLAUDE_FILE_PATH` | Path of the target file |
| `$CLAUDE_TOOL_NAME` | Name of the tool that ran |
| `$CLAUDE_TOOL_INPUT` | Input passed to the tool (JSON) |

## Real-World Scenarios

### 1. Auto-Format on File Save

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**Effect**: Every time Claude writes a file, Prettier automatically formats the code. In no-code terms, this is like "auto-sort on document save."

### 2. Block Dangerous Commands

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -q 'rm -rf'; then echo 'BLOCKED: rm -rf is not allowed' >&2; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

**Effect**: Automatically blocks Claude from running dangerous commands like `rm -rf`.

### 3. Task Completion Notification

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"$CLAUDE_NOTIFICATION_MESSAGE\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**Effect**: When Claude finishes a long task, a macOS notification pops up. You won't miss the completion even while doing something else.

### 4. Lint Check on File Change

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint \"$CLAUDE_FILE_PATH\" --fix 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

## Skills vs. Hooks — Different Approaches to Automation

| | **Skill** | **Hooks** |
|---|---|---|
| **Trigger** | User invokes with `/command` | Fires automatically on event |
| **Defined as** | Markdown (SKILL.md) | JSON config + shell command |
| **Executor** | Claude reads instructions and acts | Shell executes the command directly |
| **Best for** | Complex multi-step tasks | Simple, repetitive automatic processing |
| **Analogy** | Cooking recipe (you start it) | Sensor + automatic sprinkler (fires when condition met) |
| **Example** | "/weekly-report" | "File saved → auto-format" |

## Summary for No-Code Tool Users

If you have used Zapier or Make, think of it this way:

```
No-code tool structure:
  Trigger (detect event) → Action (execute) → Result

Claude Code Hooks structure:
  Event (before/after tool runs) → Shell Command (execute) → Result passed to Claude
```

The difference is that the **action is a shell command**. Instead of clicking in a GUI, you write a terminal command. The underlying principle is identical: "When X happens, automatically do Y."

## Key Takeaways

- Hooks = **event-driven automatic execution** (shell commands run before/after tool use)
- Four timing points: PreToolUse, PostToolUse, Notification, Stop
- Configured as JSON in `settings.json`
- Skills (manual invocation) + Hooks (automatic execution) together enable complete automation
- Mirrors the Trigger → Action pattern from no-code tools
