# Skills vs. Rules — A Decision Guide

## Why This Distinction Matters

There are four main ways to give Claude instructions in Claude Code:

```
┌─────────────────────────────────────────────────────────┐
│              Four Ways to Instruct Claude                │
├──────────────┬──────────────────────────────────────────┤
│  Always on   │  CLAUDE.md (rules)         ← loaded every session │
│              │  Memory                    ← auto-referenced      │
├──────────────┼──────────────────────────────────────────┤
│  On demand   │  Skill (.claude/skills/)   ← natural language match│
│              │  Command (~/.claude/commands/) ← /command         │
└──────────────┴──────────────────────────────────────────┘
```

Getting this wrong causes problems:
- Putting something in a Skill that should be a rule → you have to remember to invoke it every time
- Putting something in a rule that should be a Skill → CLAUDE.md bloats and wastes tokens

---

## The Core Decision Criteria

### One-Line Principle

> **"Is this a behavior principle that must always apply?"** → CLAUDE.md rule
> **"Is this a procedure I follow the same way every time it comes up?"** → Skill / Command

### Decision Flow

```
This instruction is needed
       │
       ▼
Should it apply automatically every time?
       │
  ┌────┴────┐
  YES       NO
  │         │
  ▼         ▼
Rule       Is the procedure well-defined?
(CLAUDE.md)    │
          ┌───┴───┐
          YES     NO
          │       │
          ▼       ▼
        Skill/   Just handle it
        Command  in conversation
```

### Criteria Table

| Criteria | → Rule (CLAUDE.md) | → Skill / Command |
|------|-------------------|--------------|
| **Frequency** | Every session, every response | Only at specific moments (before deploy, at session end) |
| **Judgment required** | No (follow unconditionally) | Yes (user decides when to invoke) |
| **Procedure complexity** | One line to a few lines | Multiple steps, agent combinations |
| **Context-dependent** | No (always the same) | Yes (inputs vary by situation) |
| **Output** | None (changes behavior) | Yes (report, document, checklist) |

---

## Real Examples: Decisions Made in Practice

### Case 1: "Read the front-end code first" → Rule

Initially considered making this a `/frontend-prompt` Skill, but ultimately decided on a **CLAUDE.md rule**.

```
IMPORTANT: For any front-end discussion, always read the git log
and component structure of vista-sphere-pro before responding. Never guess.
```

**Why a rule?**
- Having to invoke `/frontend-prompt` every time you say "fix the front end" is tedious
- Whenever a front-end topic comes up, the code must **always** be read first
- The user shouldn't have to decide whether to invoke it — it applies unconditionally
- It's a behavioral principle, not a defined procedure

### Case 2: "Wrap up at session end" → Skill (/wrap)

```
4 parallel agents → deduplicate → user selects → execute
```

**Why a Skill?**
- Only needed **at session end**, not every session
- It has a **defined procedure**: 4 agents running in parallel
- The user needs to **choose which updates to apply** based on the output
- It produces a clear output (work summary, next steps, update suggestions)

### Case 3: "Code and commits in English" → Rule

```
IMPORTANT: All technical artifacts — code, commit messages, CLI output —
must be in English without exception.
```

**Why a rule?**
- Applies **without exception** to all code writing and all commits
- No judgment required (always English)
- It's a principle, not a procedure

### Case 4: "6-point check before deployment" → Skill (/preflight)

```
Git → Env vars → DB migrations → CORS → Health → API changes → PASS/FAIL
```

**Why a Skill?**
- Only needed when deploying (not daily)
- Checking 6 items in order is a **well-defined procedure**
- The user **decides whether to push** based on the result
- It produces a PASS/FAIL checklist output

---

## Command vs. Skill — Where Does It Live?

Once you've decided to create a Skill/Command, determine where to put it:

```
Is this Skill useful across all projects?
       │
  ┌────┴────┐
  YES       NO
  │         │
  ▼         ▼
Command    Skill
(~/.claude/  (project/.claude/
 commands/)   skills/)
```

| Placement | Location | Invocation | Examples |
|------|------|------|------|
| **Global Command** | `~/.claude/commands/wrap.md` | `/wrap` (from anywhere) | `/wrap`, `/switch`, `/weekly` |
| **Project Skill** | `project/.claude/skills/preflight/SKILL.md` | `/preflight` (that project only) | `/preflight` |

### Additional Differences

| | Command | Skill |
|---|---------|-------|
| **Frontmatter** | Not required | `name`, `description` required |
| **Natural language trigger** | No (slash command only) | Yes (matches via description) |
| **Passing arguments** | `$ARGUMENTS` variable | Not supported |
| **File structure** | Single `.md` file | Folder + `SKILL.md` (+ references/) |

---

## CLAUDE.md Level — Which CLAUDE.md?

Once you've decided something is a rule, determine which level of CLAUDE.md it belongs in:

```
~/.claude/CLAUDE.md              ← applies to all projects (persona, language, tone)
project/CLAUDE.md                ← applies to this project only (infra, conventions)
```

| Rule Content | Level | Reason |
|-----------|------|------|
| "Respond in English" | Global | Same for all projects |
| "Pushing to main = production deploy" | Project | Specific to this project |
| "Never output .env contents" | Global | Universal security rule |
| "Read front-end code first" | Project | Specific to this project's front end |
| "Show SQL before DB changes" | Global | Universal safety rule |

---

## Practice: Make the Call

Decide whether each situation calls for a rule or a Skill, and where it belongs:

| Situation | Decision | Reason |
|------|------|------|
| "Create a change summary on every PR" | Command `/pr-summary` | Defined procedure + all projects |
| "This project uses pnpm only" | Project CLAUDE.md | Always applies + project-specific |
| "On error, check logs first" | Global CLAUDE.md | Always applies + all projects |
| "Generate a weekly cost report" | Project Skill | Defined procedure + project-specific |
| "Don't commit without tests" | Project CLAUDE.md | Always applies + test method varies by project |

---

## Summary

```
An instruction is needed
    │
    ├─ Always applies? ──→ Rule (CLAUDE.md)
    │                        ├─ All projects? → Global CLAUDE.md
    │                        └─ This project only? → Project CLAUDE.md
    │
    └─ At specific times? ──→ Skill / Command
                               ├─ All projects? → Command (~/.claude/commands/)
                               └─ This project only? → Skill (.claude/skills/)
```
