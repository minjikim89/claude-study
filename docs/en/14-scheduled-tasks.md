# Scheduled Tasks — A Scheduling System for Recurring Work

## What Are Scheduled Tasks?

If you find yourself repeatedly asking Claude to summarize your Slack messages every morning, or compile a project report every Friday — you can now **set it up once and let it run automatically**.

Scheduled Tasks are a recurring execution system built into Claude Desktop's **Cowork** feature. Write the instructions once, and Claude runs them on the schedule you set, saving the results each time.

> Summary: Scheduled Tasks = **a cron job you set up for Claude**

## Comparison with No-Code Tools

| No-Code Tool | Scheduled Tasks Equivalent |
|---|---|
| **Zapier Schedule Trigger** | Time-based auto-execution (hourly, daily, weekly) |
| **Make Scenario scheduling** | Periodic scenario execution |
| **Google Apps Script time-based trigger** | Script runs at a set time |
| **n8n Cron Node** | Repeating execution based on cron expressions |

Key difference:
- No-code tools repeat **a pre-built workflow**
- Scheduled Tasks repeat **natural language instructions** — no workflow design needed, just write what you want

## Where Does It Run?

```
┌─ Claude Desktop (Cowork) ─────────────────────┐
│                                                │
│  Scheduled Task: "Every day at 9 AM"           │
│       │                                        │
│       ▼                                        │
│  Claude runs as an independent Cowork session  │
│       │                                        │
│       ├── MCP servers accessible (Slack, Drive)│
│       ├── Plugins/Skills available             │
│       └── Results saved to the session         │
│                                                │
│  Note: computer must be on + app must be open  │
└────────────────────────────────────────────────┘
```

**Important limitation**: Scheduled Tasks run **on your computer**. If your computer is asleep or Claude Desktop is closed, the task is skipped — and runs automatically the next time you're back online.

This is a fundamental difference from Zapier/Make:
- Zapier/Make: runs in the cloud → unaffected by your computer's state
- Scheduled Tasks: runs locally → the app must be running

## Real-World Scenarios

### 1. Daily Morning Briefing
```
Every day at 9 AM
→ Summarize messages from key Slack channels
→ Organize today's calendar events
→ Remind about unresolved issues from yesterday
```

### 2. Weekly Report
```
Every Friday at 5 PM
→ Collect document changes from Google Drive this week
→ Format spreadsheet data into a report
→ Save to a designated folder
```

### 3. Competitor / Industry Trend Tracking
```
Every day
→ Search news for specific keywords
→ Summarize key findings
→ Highlight any notable trend changes
```

### 4. File Organization
```
Every Monday
→ Categorize files in the Downloads folder by type
→ Build a list of temp files older than 30 days for cleanup
```

## Two Ways to Create a Scheduled Task

### Method 1: `/schedule` Command (Conversational)

Type `/schedule` in Cowork and Claude will guide you through the setup in conversation.

```
User: /schedule
Claude: What task would you like to schedule?
User: Summarize Slack for me every morning
Claude: Choose a frequency → [Hourly] [Daily] [Weekly] [Weekdays only]
...
→ Click "Schedule" to confirm
```

### Method 2: Scheduled Tasks Page (Direct Input)

Click **"Scheduled"** in the left sidebar → click **"+ New task"**

| Field | Description |
|---|---|
| Task name | Name for the task (e.g., "Daily Slack Briefing") |
| Description | Short description |
| Prompt | Detailed instructions (written in natural language, like a Skill) |
| Frequency | Hourly / Daily / Weekly / Weekdays only / Manual |
| Model | Model to use (optional) |
| Working folder | Working directory (optional) |

## Management Features

From the Scheduled Tasks page, you can:

- View execution history (success / skipped / failed)
- Edit instructions or frequency
- Pause / resume
- Run immediately (manual trigger)
- Delete

## Requirements

| Item | Requirement |
|---|---|
| Plan | Pro, Max, Team, or Enterprise |
| Platform | Claude Desktop only (not available on web) |
| Execution conditions | Computer awake + Claude Desktop running |

## Key Takeaways

- Scheduled Tasks = a cron job that **runs natural language instructions on a schedule**
- No need to design a workflow like in Zapier/Make — just write what you want in plain language
- Runs **locally**, so your computer must be on — that's the key limitation
- Works with MCP servers, Plugins, and Skills → useful for automating external service integrations
- For 24/7 cloud-based automation, Zapier/Make is still the right tool
