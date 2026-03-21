# MCP + API Hybrid Pattern — When MCP Can't Do the Job

## The Problem

MCP servers don't wrap **every** API endpoint of an external service. For example, the Notion MCP supports page creation, editing, and search — but **archiving (deletion) is not supported**. The capability exists in the official REST API, but it's missing from the MCP server.

Why these gaps exist:
- MCP server authors don't implement every API endpoint
- Dangerous operations (delete, permission changes, etc.) are intentionally excluded
- There's a lag between service API updates and MCP server updates

## The Concept in No-Code Terms

| No-Code Tool | MCP + API Hybrid Equivalent |
|---|---|
| **Zapier**: built-in actions + Webhooks by Zapier | MCP tools + direct API calls via Bash/Python |
| **Make**: built-in modules + HTTP module | MCP tools + supplemented by curl/requests |
| **n8n**: dedicated nodes + HTTP Request node | MCP + Skill wrapping the API call |

The principle is the same: **handle 80% with built-in features, fill the remaining 20% with generic HTTP calls**.

## Three Practical Patterns

### Pattern 1: MCP First, API for the Rest (Most Common)

```
Everyday tasks (80%)              Special tasks (20%)
  ├─ Create page                    ├─ Delete page (archive)
  ├─ Query DB                       ├─ Change permissions
  ├─ Update page                    ├─ Bulk operations
  ├─ Search                         └─ Advanced filtering
  │
  └→ Handled by MCP tools           └→ Handled by direct API calls
```

**Pros**: Maximizes MCP's convenience (Claude calls tools via natural language) while patching gaps
**Cons**: Two different approaches in play — usage is inconsistent

### Pattern 2: Go All-In on Direct API Calls

Skip MCP entirely and handle everything with API call scripts.

**When to choose this**:
- When MCP supports too few features
- When fine-grained error handling is required
- When building a large-scale batch automation pipeline

**Cons**: Loses Claude's tool integration benefit. The convenience of saying "search in Notion" and having Claude pick the right MCP tool automatically disappears.

### Pattern 3: Fill the Gap with a Custom MCP Server

Wrap the missing API endpoints in your own MCP server so Claude can call everything as a tool.

```
Official Notion MCP              Your supplemental MCP
  ├─ create-page                   ├─ archive-page (delete)
  ├─ update-page                   ├─ bulk-update
  ├─ search                        └─ set-permissions
  └─ query-database
```

**Pros**: Cleanest experience (all capabilities available as tools)
**Cons**: Requires building an MCP server (Python/TypeScript) and ongoing maintenance

### How to Choose

```
There's something MCP doesn't support
  │
  ├─ Needed occasionally (1–2× per month)
  │   └→ Pattern 1: just make the API call when needed
  │
  ├─ Needed frequently (several times per week)
  │   └→ Pattern 1 + Skill: wrap the API call in a Skill for reuse
  │
  └─ Part of a core workflow
      └→ Pattern 3: integrate into a custom MCP server
```

## Implementation Example: Notion MCP + API Hybrid

### What Notion MCP Supports vs. Does Not

| Task | MCP | API | Notes |
|------|:--------:|:--------:|------|
| Create page | O | O | |
| Update page | O | O | |
| Search pages | O | O | |
| Query DB | O | O | |
| Get/create comments | O | O | |
| Archive page (delete) | X | O | PATCH `archived: true` |
| Permanently delete page | X | X | Not supported by API either |
| Delete DB property | X | O | PATCH property to `null` |
| Delete block | X | O | block DELETE endpoint |

### Scenario: "Archive completed tasks"

This workflow can't be done with MCP alone:
1. **Query completed tasks from DB** → possible with MCP (query-database)
2. **Archive each task** → not supported by MCP → direct API call required

### Implementation A: Ask Claude to Make the API Call Directly

No extra setup needed — just ask Claude in natural language:

```
Query the Tasks DB in Notion for items where Done is true,
then archive each page using the Notion API.
Use the NOTION_API_KEY environment variable for the API key.
```

What Claude executes:

```
Step 1: Query DB via MCP tool
  → notion.query-database (filter: Done = true)
  → Result: list of page_ids

Step 2: Direct API call via Bash (MCP doesn't support this)
  → curl -X PATCH https://api.notion.com/v1/pages/{page_id} \
      -H "Authorization: Bearer $NOTION_API_KEY" \
      -H "Notion-Version: 2022-06-28" \
      -d '{"archived": true}'
```

**Pros**: Works immediately, no setup required
**Cons**: You have to re-explain the API key location and call method every time

### Implementation B: Wrap in a Skill for Reuse (Recommended)

Turn frequently used hybrid tasks into a Skill — no re-explanation needed, just invoke with a slash command.

```
.claude/skills/
  └─ notion-archive/
      └─ SKILL.md
```

**SKILL.md example:**

```markdown
---
name: notion-archive
description: Archive completed tasks in Notion. Triggered by "archive" or "clean up completed" requests.
---

# Notion Completed Task Archive

## Environment
- NOTION_API_KEY: read from environment variable
- Tasks DB ID: 6d1161325e944014b3c369983796ef15

## Execution Steps

### Step 1: Query completed tasks (via MCP)
Use Notion MCP's query-database tool to fetch items where Done = true from the Tasks DB.

### Step 2: Confirm archiving
Show the list of queried tasks to the user and confirm before proceeding.
Archiving is irreversible — always confirm first.

### Step 3: Archive via API
Call the Notion REST API for each page to set archived: true.

curl -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"archived": true}'

### Step 4: Report results
Output success/failure counts as a table.
```

**Usage:**

```
/notion-archive
or
"clean up completed tasks"
```

### Implementation C: Separate Python Script

When the logic is complex or runs frequently, extract it into a standalone script:

```python
# scripts/notion_archive.py
import os
import requests

NOTION_API_KEY = os.environ["NOTION_API_KEY"]
TASKS_DB_ID = "6d1161325e944014b3c369983796ef15"
HEADERS = {
    "Authorization": f"Bearer {NOTION_API_KEY}",
    "Notion-Version": "2022-06-28",
    "Content-Type": "application/json",
}

def query_done_tasks():
    """Query tasks where Done = true"""
    url = f"https://api.notion.com/v1/databases/{TASKS_DB_ID}/query"
    payload = {
        "filter": {
            "property": "Done",
            "checkbox": {"equals": True}
        }
    }
    resp = requests.post(url, headers=HEADERS, json=payload)
    resp.raise_for_status()
    return resp.json()["results"]

def archive_page(page_id: str):
    """Archive a page"""
    url = f"https://api.notion.com/v1/pages/{page_id}"
    resp = requests.patch(url, headers=HEADERS, json={"archived": True})
    resp.raise_for_status()
    return resp.json()

def main():
    tasks = query_done_tasks()
    print(f"Found {len(tasks)} completed tasks")

    for task in tasks:
        title = task["properties"]["Name"]["title"][0]["plain_text"]
        result = archive_page(task["id"])
        status = "archived" if result.get("archived") else "failed"
        print(f"  {status}: {title}")

if __name__ == "__main__":
    main()
```

Invoke this script from a Skill:

```markdown
## Execution
Run python scripts/notion_archive.py
```

## Pattern Comparison

| Criteria | A: Direct Request | B: Skill Wrapper | C: Separate Script |
|------|:-----------:|:------------:|:---------------:|
| Initial setup | None | 1 Skill file | Skill + Python file |
| Reusability | Low (re-explain every time) | High (slash command) | High (script execution) |
| Error handling | Claude's discretion | Can specify in Skill | Precise control in code |
| Best for | One-off tasks | Recurring periodic tasks | Complex logic, batch processing |

## Key Takeaways

- MCP only supports a **subset** of an external service's API — it doesn't cover everything
- The most common approach is a **hybrid pattern** that supplements gaps with **direct API calls (curl, Python requests)**
- Wrap frequently used hybrid tasks in a **Skill** so they're reusable without re-explanation every time
- For complex logic or precise error handling, **extract into a separate script**
- MCP's limitations are exactly **why building your own Skills matters**. Filling the gap in generic tools with your own workflow is what real automation looks like
