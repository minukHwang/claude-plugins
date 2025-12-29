# Daily Plugin

Morning daily planning with Notion TODO and Jira integration.

## Version

1.1.0

## What's New in v1.1.0

- **Carryover tracking**: Shows 🔄 indicator with postponement count
- Items carried over increment Carryover field (+1)

## Prerequisites

- Notion MCP server configured
- `/workflow:init` completed with Notion enabled

## Commands

| Command | Description |
|---------|-------------|
| `/daily:plan` | Morning daily planning |

## Features

### Daily Planning Flow

```
1. Ask today's tasks
2. Review yesterday (completion check)
3. Carry over incomplete tasks
4. AI time blocking recommendation
5. Save to Notion TODO
6. Optional: Sync to Jira
```

### Review Since Last Planning

- Shows items since last planning date
- Displays completion status (Done / Incomplete)
- AI feedback based on completion rate

### Carryover

- Lists incomplete tasks since last planning
- Shows 🔄 indicator with postponement count
- Multi-select items to bring to today
- Updates Period to today
- Increments Carryover count (+1)

### Time Blocking

- AI recommends schedule based on task types
- User can confirm or modify
- Saves to Period field (datetime)
- Shows in Notion Calendar

### Jira Integration

At the end of planning:
- "Sync to Jira?" prompt
- If yes → runs `/jira:sync`
- Syncs TODO items with Jira issues

## Notion Databases

### [CLAUDE] TODO (existing)

Uses existing TODO database from `/workflow:init`.

| Field | Type | Description |
|-------|------|-------------|
| Title | title | Task name |
| Type | select | Todo/Task/Story/Bug/Epic |
| Status | status | Todo/In Progress/Done |
| Period | date | Start → End (datetime for calendar) |
| Carryover | number | Postponement count (0 = never postponed) |

## Configuration

### User Config: `~/.claude/notion.json`

```json
{
  "todo": {
    "id": "xxx",
    "pageId": "yyy",
    "name": "[CLAUDE] TODO"
  }
}
```

## Workflow Example

```
/daily:plan
    |
    v
"What are you planning to do today?"
→ "API work, grocery shopping, exercise"
    |
    v
Yesterday Review:
"You completed 3 out of 4 yesterday! Nice!"
    |
    v
Incomplete items:
- [ ] Bug fix (yesterday) 🔄 x1
→ Select to carry over
    |
    v
AI Schedule:
├─ 09:00-12:00 | API work
├─ 13:00-14:00 | Bug fix
├─ 17:00-18:00 | Grocery shopping
└─ 19:00-20:00 | Exercise
→ "Looks good" / "Modify"
    |
    v
Saved to Notion TODO
    |
    v
"Sync to Jira?"
→ Yes → /jira:sync
    |
    v
✓ Today's plan is set!
```

## Requirements

- Notion MCP (mcp__notion__*)
- workflow plugin configured
- jira plugin (optional, for sync)
