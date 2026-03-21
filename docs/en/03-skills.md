# Skills — Automation Recipes in Markdown

## What Is a Skill?

A Skill is Claude Code's mechanism for **automating repetitive tasks**. The concept is simple: **one markdown file equals one Skill**.

No code to write, no complex configuration — just write instructions in markdown telling Claude "do it this way," and Claude follows them.

## Comparison with No-Code Tools

| No-code tool (Zapier, Make, etc.) | Claude Code Skill |
|---|---|
| Build workflows with drag-and-drop in a GUI | Write instructions as text in a markdown file |
| Visually design Trigger → Action → Result flows | Claude reads the instructions and executes them in order |
| Separate settings UI for each action | Describe what you want in plain language |
| Requires specific app integrations (API keys, etc.) | Uses Claude's built-in tools (file read/write, terminal, etc.) |

> If no-code tools are "assembling LEGO bricks," a Skill is closer to "handing someone a recipe card."

## Under the Hood: Prompt Injection

The phrase "run a Skill" is used, but **no separate program actually executes**.

```
1. User types "/day1-test-skill"
2. Claude Code finds .claude/skills/day1-test-skill/SKILL.md
3. Injects its content into the context window (appended to the system prompt)
4. Claude reads the instructions and responds accordingly
```

In other words, "running" a Skill is really **"making Claude read a set of instructions."** No separate agent is created — it is **the same Claude receiving additional directives**.

## Skill File Structure

Skills live in the `.claude/skills/` directory inside your project.

```
project/
└── .claude/
    └── skills/
        ├── sync-report/
        │   ├── SKILL.md           ← Main instructions
        │   ├── references/        ← Reference data (optional)
        │   │   └── template.md
        │   └── scripts/           ← Execution scripts (optional)
        │       └── gather-data.sh
        ├── daily-standup/
        │   └── SKILL.md
        └── code-review/
            └── SKILL.md
```

### Example SKILL.md

```markdown
# Daily Standup Summary

## Role
You are a project manager.

## Steps
1. Check today's git commit log
2. Analyze changed files
3. Format the output as follows:

### Done Today
- [item]

### Tomorrow's Plan
- [item]

### Blockers
- [item]
```

## Skills vs. CLAUDE.md

Both give Claude instructions via markdown, but they serve different purposes.

| | **CLAUDE.md** | **Skill** |
|---|---|---|
| **When loaded** | Automatically, every session | Only when invoked with `/command` |
| **Purpose** | Rules applied across the entire project | Recipe for a specific task |
| **Length** | Keep short (always loaded → conserve tokens) | Can be long (only loaded when needed) |
| **Analogy** | Company policy | Task manual |
| **Example** | "Reply in English", "run tests first" | "/sync", "/daily-standup" |
| **Count** | Max 3 (one per level) | Unlimited |

## Real-World Scenarios Where Skills Shine

### 1. Recurring Reports
When you need to produce the same report format every week — put the template and data-gathering instructions in a Skill, and `/weekly-report` produces it in one shot.

### 2. Code Review Checklist
When every PR needs a check for security, performance, and code style — use `/code-review` to get consistently structured reviews.

### 3. Onboarding Guide
When a new team member joins — use `/onboarding` to walk them through the project structure, environment setup, and first task.

## Key Takeaways

- Skill = **a markdown file** (not code; non-developers can write them)
- "Running" = **injecting instructions into the context** (not launching a separate program)
- CLAUDE.md = always-applied rules / Skill = a recipe you call when you need it
- For complex multi-agent workflows, use **Subagents** and **Agent Teams** rather than Skills
