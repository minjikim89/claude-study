# Plugins — An App Store for Packaging and Sharing Features

## What Is a Plugin?

Everything covered so far — Skills, Hooks, MCP servers, and Agents — existed as individual files inside the `.claude/` directory. That works fine, but **sharing them with other projects or teammates required copying files manually**.

Plugins solve this. A Plugin **bundles multiple capabilities (Skills + Hooks + MCP + Agents) into a single folder as a package, which can then be installed and distributed through a Marketplace**.

> Summary: Plugin = feature bundle package + app store distribution

## Comparison with No-Code Tools

| No-Code Tool | Plugin Equivalent |
|---|---|
| **Zapier Template** | Installing a Plugin someone else built (`/plugin install`) |
| **Notion Template Gallery** | Plugin Marketplace (official + community) |
| **Chrome Web Store extension** | Plugin package (Skills + Hooks + MCP bundled) |
| **WordPress plugin** | Claude Code Plugin (install → add features → enable/disable) |

Key difference: No-code tools let you **click in a marketplace inside the app**, while Claude Code uses **`/plugin` commands to install and manage** them.

## Standalone vs. Plugin — When to Use Which

| | **Standalone** (direct `.claude/` setup) | **Plugin** (packaged) |
|---|---|---|
| **Skill name** | `/hello` | `/plugin-name:hello` |
| **Best for** | Personal use, project-specific, quick experiments | Team sharing, community distribution, multi-project reuse |
| **Management** | Manual file copying | Version control + automatic updates |
| **Analogy** | A macro saved on your own computer | An official tool used company-wide |

> **Tip**: Build and test in `.claude/` first — convert to a Plugin when you need to share. That's the most natural flow.

## Plugin Structure

A Plugin puts everything inside **one folder**.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          ← metadata (name, description, version)
├── commands/                ← slash commands (markdown files)
├── skills/                  ← Agent Skills (SKILL.md)
├── agents/                  ← custom agent definitions
├── hooks/
│   └── hooks.json           ← event hook configuration
├── .mcp.json                ← MCP server configuration
├── .lsp.json                ← LSP server configuration (code intelligence)
└── settings.json            ← default settings
```

### plugin.json — The Plugin's Identity

```json
{
  "name": "my-plugin",
  "description": "Code review automation plugin",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

| Field | Purpose |
|---|---|
| `name` | Unique identifier (= namespace prefix on Skill names) |
| `description` | Description shown in the Marketplace |
| `version` | Version tracking (used for update detection) |

> **Note**: `commands/`, `skills/`, `agents/`, etc. should be at the **Plugin root** — outside the `.claude-plugin/` directory. Only `plugin.json` lives inside `.claude-plugin/`.

## Building a Plugin — Minimal Example

### 1. Create directories

```bash
mkdir -p my-plugin/.claude-plugin
mkdir -p my-plugin/skills/review
```

### 2. Write plugin.json

```json
// my-plugin/.claude-plugin/plugin.json
{
  "name": "review-plugin",
  "description": "Code review automation",
  "version": "1.0.0"
}
```

### 3. Write a Skill

```markdown
<!-- my-plugin/skills/review/SKILL.md -->
---
description: Review code for bugs, security, and performance
disable-model-invocation: true
---

Review the code for:
1. Potential bugs or edge cases
2. Security concerns
3. Performance issues
```

### 4. Test locally

```bash
claude --plugin-dir ./my-plugin
```

Inside Claude Code, run `/review-plugin:review`.

## Marketplace — The Plugin App Store

The Marketplace is a Plugin catalog. It's a two-step flow: **add a store → install the Plugin you want**.

### Official Marketplace

When you start Claude Code, the **official Anthropic Marketplace** is automatically registered. Open `/plugin` and browse plugins directly from the **Discover** tab.

### Key Plugin Categories in the Official Marketplace

**1. Code Intelligence**

Connects language servers (LSP) so Claude can immediately check type errors, jump to definitions, and search references.

| Language | Plugin Name | Required Binary |
|---|---|---|
| TypeScript | `typescript-lsp` | `typescript-language-server` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Go | `gopls-lsp` | `gopls` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |

**2. External Integrations**

Plugins with MCP servers pre-configured — no manual setup required.

- GitHub, GitLab (source control)
- Slack (communication)
- Notion, Linear, Jira (project management)
- Figma (design)
- Vercel, Supabase (infrastructure)

**3. Development Workflows**

- `commit-commands` — Git commit/push/PR workflow
- `pr-review-toolkit` — PR review agent
- `plugin-dev` — Plugin development tools

## Plugin Install and Management Flow

```
/plugin                              ← Open Plugin manager UI
├── Discover tab: browse installable Plugins
├── Installed tab: manage installed Plugins
├── Marketplaces tab: add/remove Marketplaces
└── Errors tab: view errors
```

### Basic Commands

```bash
# Add a Marketplace (GitHub repo)
/plugin marketplace add owner/repo

# Install a Plugin
/plugin install plugin-name@marketplace-name

# Disable a Plugin (turn off without uninstalling)
/plugin disable plugin-name@marketplace-name

# Re-enable a Plugin
/plugin enable plugin-name@marketplace-name

# Fully uninstall a Plugin
/plugin uninstall plugin-name@marketplace-name
```

### Installation Scope

| Scope | Who Uses It | Stored In |
|---|---|---|
| **User** | Only me, across all projects | User settings |
| **Project** | All collaborators on this repo | `.claude/settings.json` |
| **Local** | Only me, in this repo only | Local settings |

## Converting Existing Configuration to a Plugin

If you already have Skills or Hooks inside `.claude/`, converting them to a Plugin is straightforward:

```bash
# 1. Create Plugin directory
mkdir -p my-plugin/.claude-plugin

# 2. Write plugin.json
# { "name": "my-plugin", "version": "1.0.0", ... }

# 3. Copy existing files
cp -r .claude/commands my-plugin/
cp -r .claude/skills my-plugin/

# 4. Test
claude --plugin-dir ./my-plugin
```

| Before (Standalone) | After (Plugin) |
|---|---|
| Used in this project only | Shareable via Marketplace |
| Files in `.claude/commands/` | Files in `plugin-name/commands/` |
| Share by manual file copy | Install with `/plugin install` |

## Running a Team Marketplace

To give your whole team the same set of Plugins, register the Marketplace in the project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "github",
        "repo": "our-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "formatter@team-tools": true,
    "reviewer@team-tools": true
  }
}
```

When a teammate opens the project, they'll automatically see a prompt to install from the Marketplace.

## Relationship to Previous Concepts

```
Plugin = packaging + distribution layer
  ├── Skills (03)     ← included as skills/ directory inside Plugin
  ├── Hooks (08)      ← included as hooks/hooks.json inside Plugin
  ├── MCP (07)        ← included as .mcp.json inside Plugin
  ├── Agents (04, 05) ← included as agents/ directory inside Plugin
  └── LSP (new)       ← Plugin-exclusive: code intelligence server integration
```

Skills, Hooks, and MCP servers you've already learned are all **components of a Plugin**. A Plugin adds a **packaging and distribution layer** that bundles them together.

## Key Takeaways

- Plugin = existing features (Skills + Hooks + MCP + Agents) **packaged into a single folder**
- Marketplace = an app store to **discover, install, and update** Plugins
- Manage with `/plugin` commands — install, disable, uninstall
- Build and experiment in `.claude/` first → convert to a Plugin when sharing is needed
- Set up a team Marketplace to automatically distribute standard tools to everyone on the team
- Minimum version required: **Claude Code v1.0.33+**
