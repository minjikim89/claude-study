# Memory System — How Claude Remembers

## The Baseline: Claude Forgets Everything When a Session Ends

If you have used ChatGPT or the Claude web interface, you already know: AI fundamentally **cannot recall previous conversations once a session ends**. Every session starts as if meeting for the first time.

Claude Code addresses this fundamental limitation through a **file-based memory system**. Think of it as "a secretary with no innate memory who compensates by taking meticulous notes."

## Memory Classified by Lifespan

### Volatile Memory (gone when the session ends)

| Type | Description | Analogy |
|---|---|---|
| **Conversation (Messages)** | Messages exchanged right now | A phone call in progress |
| **Loaded Skill content** | Instructions loaded by `/command` | A manual you currently have open |

When you close the session, this content is gone entirely. Ask "do you remember what we discussed earlier?" in the next session — Claude won't know.

### Persistent Memory (saved to files, survives across sessions)

| Type | Description | Analogy |
|---|---|---|
| **CLAUDE.md** | Rules written by the user | Company policy document |
| **Auto Memory** | Notes Claude automatically writes | The assistant's personal notebook |
| **Skill files** | Task recipes stored on disk | A recipe book |

These are markdown files on disk, so they persist across sessions unchanged.

## Comparing the Three Persistent Memory Types

| | **CLAUDE.md** | **Auto Memory** | **Skill** |
|---|---|---|---|
| **What it is** | Rulebook | Learning notes | Task recipe |
| **Analogy** | Company policy | Assistant's personal notebook | Cookbook |
| **Who writes it** | Human (manual) | Claude (auto) + Human | Human (manual) |
| **When loaded** | Every session, automatically | Every session, automatically | Only when invoked |
| **Stored at** | Project root / `~/.claude/` | `~/.claude/projects/<project>/memory/` | `.claude/skills/` |
| **Purpose** | "Do it this way" | "I learned this" | "This task goes like this" |
| **Example** | "Reply in English" | "This user prefers tables" | "/sync generates a report" |
| **Edit rights** | User only | Claude auto + User manual | User only |

## Auto Memory in Detail

### How It Works

Auto Memory is Claude's ability to **automatically note patterns it discovers during conversation**.

```
Claude notices a pattern during conversation
  │
  ▼
"This user prefers table format"
  │
  ▼
Automatically written to ~/.claude/projects/<project>/memory/MEMORY.md
  │
  ▼
Read automatically in the next session → responds with tables from the start
```

### What Auto Memory Records

- **User preferences**: response format, language, coding style
- **Project patterns**: frequently used file paths, architecture decisions
- **Debugging insights**: solutions to recurring problems
- **Workflows**: the user's preferred order of operations

### What Auto Memory Does Not Record

- Session-scoped context (work in progress, temporary state)
- Unverified guesses
- Content that duplicates CLAUDE.md

### Viewing and Editing

```bash
# View and edit Auto Memory with the /memory command
# Run inside Claude Code

/memory
```

You can also open the file directly:
```
~/.claude/projects/<project-name>/memory/MEMORY.md
```

### Comparison with No-Code Tools

| No-code tool | Claude Code Auto Memory |
|---|---|
| Notion AI's "learn my writing style" | Learns user preference patterns |
| Zapier's history-based recommendations | Carries learnings from past sessions into new ones |
| ChatGPT's "Memory" feature | Nearly identical concept, but file-based and transparent |

The key difference from ChatGPT Memory: Claude Code's Auto Memory is a **markdown file** you can directly read and edit. It is not a black box.

## One-Line Summary

```
CLAUDE.md  = Rules you give Claude          (always loaded)
Memory     = Notes Claude writes itself     (always loaded)
Skill      = Recipe for a specific task     (loaded on demand)
Conversation = Context of this moment       (gone when session ends)
```

> These four layers are ultimately a **design for efficient use of the context window (200k tokens)**.
> Always-needed things auto-load. Occasionally-needed things load on demand. Ephemeral things exist only in the conversation.

## Key Takeaways

- Claude has no innate memory → the file system compensates for this
- Two tiers: volatile (conversation) vs. persistent (CLAUDE.md, Auto Memory, Skill)
- Auto Memory records Claude's in-session learnings to a file → applied in the next session
- All memory is markdown, so it can be transparently viewed and edited
- Efficient use of the 200k-token context window is the core design goal of the memory system
