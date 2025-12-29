---
description: Morning daily planning
---

# Daily Plan

Plan your day in the morning. Review yesterday, carry over incomplete tasks, and schedule today.

## Step 0: Load Configuration

### 0.0 Get Current Date and Time

```bash
date "+%Y-%m-%d %H:%M %A"
```

Example output: `2025-01-15 09:30 Wednesday`

Store:
- `{today}` - Current date (YYYY-MM-DD)
- `{current_time}` - Current time (HH:MM)
- `{day_of_week}` - Day of week

Use this to:
- Filter items since last planning
- Suggest realistic time blocks (skip past hours)
- Show remaining hours in the day

### 0.1 Workflow Config

```bash
cat .claude/workflow.json 2>/dev/null
```

Check `notion.enabled`. If not exists:
```
✗ Configuration not found.
Run /workflow:init first.
```

### 0.2 Notion Config

```bash
cat ~/.claude/notion.json 2>/dev/null
```

Extract:
- `todo.id` - TODO database ID

## Step 1: Ask Today's Tasks

**Ask user (AskUserQuestion):**
"What are you planning to do today?"

User enters freely (newlines, list format, etc.).

## Step 2: Review Since Last Planning

### 2.1 Find Last Planning Date

Query TODO items with Period set, find the most recent Period:start date before today.
This is the "last planning date".

### 2.2 Query Items Since Last Planning

```
mcp__notion__notion-fetch:
  id: {todo.id}
```

Filter: Period:start >= {last_planning_date} AND Period:start < {today}

### 2.3 Check Completion Status

For each item, check Status:
- Done → Completed
- Others → Incomplete

### 2.4 AI Feedback

AI provides natural feedback based on completion:
- "You completed 4 out of 5 since last planning! Great job"
- "3 days since last planning. 2 out of 6 done."

## Step 3: Carry Over Incomplete Tasks

### 3.1 Show Incomplete Items

From Step 2, display items where Status != Done:

```
Incomplete Items (since last planning):

- [ ] API development (2 days ago)
- [ ] Grocery shopping (2 days ago)
- [ ] Bug fix (3 days ago)
```

### 3.2 Select Items to Carry Over

**If 3 or fewer items:** Show directly with multiSelect

**Ask user (AskUserQuestion, multiSelect):**
"Select items to carry over"

| Option | Description |
|--------|-------------|
| API development | 2 days ago |
| Grocery shopping | 2 days ago |
| All items | Carry all |

**If 4+ items:** Show numbered list, then ask

```
Incomplete items (7):
1. API development (2 days ago)
2. Grocery shopping (2 days ago)
3. Bug fix (3 days ago)
...
```

**Ask user (AskUserQuestion):**
"Which items to carry over?"

| Option | Description |
|--------|-------------|
| All | Carry all items |
| None | Skip all |
| Select | Enter numbers |

If "Select" → Ask "Enter item numbers (e.g., 1,3,5)"

Unselected items remain as-is.

## Step 4: Time Blocking Recommendation

### 4.1 AI Suggests Schedule

AI analyzes all tasks (new + carryover) and suggests time blocks:

```
"How about this schedule?"

Today's Schedule:
├─ 09:00-12:00 | API development
├─ 13:00-15:00 | Bug fix
├─ 15:00-16:00 | Grocery shopping
└─ 19:00-20:00 | Exercise
```

### 4.2 Confirm

**Ask user (AskUserQuestion):**

| Option | Description |
|--------|-------------|
| Looks good | Proceed |
| Modify | Adjust manually |

If modify, let user change time for each item.

## Step 5: Save to Notion

### 5.1 Create New Items

For new tasks from Step 1:

```
mcp__notion__notion-create-pages:
  parent: {"type": "data_source_id", "data_source_id": "{todo.id}"}
  pages: [{
    "properties": {
      "Title": "{task_title}",
      "Type": "Todo",
      "Status": "Todo",
      "date:Period:start": "{today}T{start_time}",
      "date:Period:end": "{today}T{end_time}",
      "date:Period:is_datetime": 1
    }
  }]
```

### 5.2 Update Carryover Items

```
mcp__notion__notion-update-page:
  data: {
    "page_id": "{item_id}",
    "command": "update_properties",
    "properties": {
      "date:Period:start": "{today}T{start_time}",
      "date:Period:end": "{today}T{end_time}",
      "date:Period:is_datetime": 1
    }
  }
```

## Step 6: Jira Sync

**Ask user (AskUserQuestion):**
"Sync to Jira?"

| Option | Description |
|--------|-------------|
| Yes | Run /jira:sync |
| No | Skip |

If Yes → Execute `/jira:sync`

## Output

```
✓ Today's plan is set!

Summary:
├─ Total: 5 items
├─ New: 3 items
└─ Carryover: 2 items

Schedule:
├─ Morning: API development
├─ Afternoon: Bug fix, Grocery shopping
└─ Evening: Exercise

Check your schedule in Notion Calendar!
```

## Constraints

1. Notion MCP connection required
2. Check notion.enabled=true in workflow.json
3. Period saved as datetime (shows time in Calendar)
4. AskUserQuestion options max 4
