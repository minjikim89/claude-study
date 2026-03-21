# Real-World Workflows — Claude Code Usage Patterns

## Purpose of This Document

The earlier documents covered individual Claude Code concepts — the system, memory, Skills, Subagents, MCP, Hooks, and more. This document focuses on **how to combine these concepts for actual work**, organized by pattern.

## Pattern 1: CLAUDE.md Writing Strategy

### Global CLAUDE.md — Personal Work Style

```markdown
# ~/.claude/CLAUDE.md

## Language
- Default response language: English
- Keep code, commit messages, and CLI output in English

## Tone
- Concise and logical tone
- Skip unnecessary pleasantries

## Preferences
- Prefer tables for structured answers
- Briefly explain the reason when making code changes
```

**Key point**: This file applies to all projects, so include only **personal preferences** like language, tone, and format. Project-specific rules go elsewhere.

### Project CLAUDE.md — Team Conventions

```markdown
# ./CLAUDE.md

## Project
- Next.js 14 + TypeScript + Tailwind CSS
- Package manager: pnpm

## Conventions
- Components: PascalCase (UserProfile.tsx)
- API routes: kebab-case (user-profile/)
- Commit messages: Conventional Commits (feat:, fix:, docs:)

## Rules
- Always include tests when adding new features
- No console.log in commits
- No any types
```

**Key point**: Rules shared by the whole team. Commit this to git and use it as a team convention.

### Common CLAUDE.md Mistakes

| Mistake | Why It's a Problem | Alternative |
|---|---|---|
| Pasting the entire project documentation | Thousands of tokens consumed every session | Keep only core rules; move details to Skills |
| Vague expressions like "if possible, please..." | Claude may ignore them | Use imperative: "Always do X" |
| Conflicting rules in multiple places | Unpredictable behavior | Define in one place, specify priority |
| Too many rules (50+) | Hard to follow all of them | Limit to the 10 most important |

## Pattern 2: Daily Work Automation

### Morning Routine Skill

```markdown
# .claude/skills/morning-check/SKILL.md

## Role
Assistant that checks project status

## Steps
1. Check commits since yesterday with git log
2. Check uncommitted work with git status
3. Search for TODO/FIXME comments
4. Summarize in the following format:

### Yesterday's Commits
- [list]

### In Progress (uncommitted changes)
- [list]

### Remaining TODOs
- [file:line] content
```

**Usage**: Run `/morning-check` every morning to get a quick project status overview.

### Comparison with No-Code Tools

| Old Approach | Claude Code Approach |
|---|---|
| Manually writing daily logs in Notion | Auto-generated with `/morning-check` |
| Checking issues on a Jira dashboard | Track actual code changes via git |
| Manually inputting Slack standup bot | Automated analysis delivered as a summary |

## Pattern 3: Learning and Documentation Workflow

### Analyzing a New Codebase

```
Step 1: Get the big picture (uses Explore Subagent automatically)
  "Analyze this project's directory structure and main tech stack"

Step 2: Trace core flows
  "Trace the code execution order when a user logs in"

Step 3: Document
  "Organize the analysis into docs/architecture.md"
```

**Key point**: Step 1 automatically uses the Explore Subagent. It explores all files in an independent space — only the summary comes back to the main context.

### Learning Note Pattern

```
Step 1: Ask about the concept
  "Explain the difference between Next.js App Router and Pages Router"

Step 2: Generate example code
  "Show the difference with a simple code example"

Step 3: Save the summary
  "Organize everything so far into docs/nextjs-routing.md"
```

## Pattern 4: Automated Code Review

### Skill + Subagent Combination

```markdown
# .claude/skills/review/SKILL.md

## Role
Code reviewer from a senior developer's perspective

## Steps
1. Analyze changes from the latest commit
2. Review from these angles:
   - Security: SQL injection, XSS, etc.
   - Performance: unnecessary computation, N+1 queries
   - Readability: naming, function length
   - Tests: missing test cases
3. Classify by severity and output as a table

## Output Format
| File | Line | Severity | Category | Detail |
```

**Advanced usage**: With Agent Teams enabled, you can run security, performance, and readability specialists as separate parallel agents.

## Pattern 5: Recurring Report Generation

### MCP + Skill Combination

```markdown
# .claude/skills/weekly-report/SKILL.md

## Steps
1. Query this week's merged PRs via GitHub MCP
2. Write a summary of each PR's changes
3. Compile issue statistics for the week (created/resolved)
4. Save the report to docs/reports/
5. (If Slack MCP configured) Send summary to #team-updates

## Report Format
# Weekly Dev Report (YYYY-MM-DD)

## Completed
| PR | Author | Summary |

## Issue Status
- New: N
- Resolved: N
- Open: N

## Next Week
- [planned items]
```

## Pattern 6: Building Safe Automation

### Hooks + Custom Subagent Combination

```
Automation layers:

1. Hooks (automatic)
   └─ On file save → run Prettier + ESLint automatically
   └─ Dangerous commands → auto-blocked

2. Custom Subagents (restricted permissions)
   └─ DB query only (Read, Grep allowed; Write/Bash blocked)
   └─ Log analysis only (access to specific directories only)

3. Skills (manual invocation)
   └─ Deployment checklist (human review before running)
```

**Key point**: Use tools in layers based on automation level. Fully automatic (Hooks) → semi-automatic (Subagent) → manual (Skill).

## Pattern 7: Using Claude Code Without a Desktop

Claude Code runs in a terminal, so it might seem impossible to use without a PC. But a few approaches make mobile access viable.

### Chat Interface vs. CLI — Know the Difference First

The claude.ai app (Chat Interface) and Claude Code (CLI) are **entirely separate products**. Conversation history, settings, and files are not shared between them (see [01-claude-code-system.md](01-claude-code-system.md)). If you want "the same Claude Code experience" on mobile, you need to **remote into the CLI itself** — not just open the claude.ai app.

### Comparison by Method

| Method | Requirements | Full CLI Features | Best For |
|------|----------|:------------:|----------|
| **Claude Mobile App** | None | No (chat only) | Quick questions, brainstorming |
| **SSH + Mobile Terminal** | PC powered on + SSH configured | Yes (full) | Continuing code work |
| **GitHub Codespaces** | GitHub account | Yes (full) | Long-term PC unavailability |

### SSH Remote Access Setup (Recommended)

The most practical combination: **Tailscale (free VPN mesh) + a mobile terminal app**

```
[Mobile]                         [PC (home/office)]
Termius / Blink Shell  ───→  SSH server (remote login enabled)
        │                              │
        └──── Tailscale VPN ───────────┘
               (no port forwarding needed, secure from anywhere)
```

Setup steps:
1. PC: System Settings → General → Sharing → Enable **Remote Login**
2. Install **Tailscale** on both PC and mobile, sign in with the same account
3. SSH from your mobile terminal app (Termius, etc.) using the IP Tailscale assigned
4. Run `claude` in the terminal → same environment as if you were at your desk

**Why Tailscale**: Regular SSH requires configuring router port forwarding, static IP, and other network setup. Tailscale automatically builds a virtual private network between devices just by installing it — you can connect to your home PC directly from a cafe, subway, or anywhere.

### Mobile Terminal App Options

| App | Platform | Notes |
|----|--------|------|
| **Termius** | iOS / Android | Clean UI, free plan is sufficient |
| **Blink Shell** | iOS | Professional-grade, mosh support |
| **JuiceSSH** | Android | Lightweight and fast |

### Keeping Context in Sync Between Chat and CLI

Since the two products don't share history, you need a manual strategy to maintain context:

| Strategy | How |
|------|------|
| **Shared knowledge base** | Organize key context in Notion, etc. Connect CLI via MCP; reference manually in Chat |
| **Session handoff** | After CLI work, save a summary to a file → paste that summary into Chat to continue |
| **Direct SSH access** | Use the CLI itself on mobile → no history problem at all (cleanest) |

## Workflow Design Guide

### Which Tool to Use When

```
"I want to automate this task"
  │
  ├─ Simple, identical processing every time? ──▶ Hooks
  │   (formatting, lint, blocking)
  │
  ├─ Recurring but requires judgment? ──▶ Skill
  │   (report generation, code review)
  │
  ├─ External service integration needed? ──▶ MCP + Skill
  │   (sending to Slack, querying DB)
  │
  ├─ Bulk data processing? ──▶ Subagent
  │   (analyzing 100 files, parsing large logs)
  │
  └─ Multiple expert perspectives needed? ──▶ Agent Teams
      (parallel review, concurrent development)
```

### No-Code → Claude Code Migration Map

| Existing No-Code Workflow | Claude Code Equivalent |
|---|---|
| Zapier: Gmail received → organized in Notion | MCP (Gmail) + Skill (organization logic) |
| Make: Every Monday → generate report → send to Slack | Skill (report) + MCP (Slack) |
| GitHub Actions: PR → auto review | Hooks (PR event) + Skill (review logic) |
| Notion automation: status change → notification | Hooks + MCP (Notion) |

## Key Takeaways

- **CLAUDE.md**: Split into layers — global (personal preferences) / project (team rules) / local (personal overrides)
- **Daily automation**: Turn repetitive tasks into recipes with Skills
- **Safe automation**: Layer design — Hooks (automatic) → Subagent (semi-automatic) → Skill (manual)
- **External integrations**: MCP + Skill combination achieves no-code-level workflows
- **Mobile access**: Even without a PC, SSH (Tailscale + terminal app) gives full remote CLI access. Chat and CLI are separate — a context-maintenance strategy is needed
- Tool selection: simple repetition (Hooks) / requires judgment (Skill) / external integration (MCP) / bulk processing (Subagent) / multiple perspectives (Teams)
