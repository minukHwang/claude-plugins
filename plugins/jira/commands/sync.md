---
description: Sync Notion TODO items to Jira (creates missing issues)
---

# Sync Notion TODO to Jira

Sync Notion TODO items (without Jira Link) to Jira as Tasks.

## Usage

```
/jira:sync
```

## Step 0: Load Configuration

### 0.1 Load Workflow Config

```bash
cat .claude/workflow.json 2>/dev/null
```

If not exists or `jira.enabled` is false:
```
✗ Jira integration not configured.
Run /workflow:init to set up Jira integration.
```

Extract:
- `jira.cloudId` - Cloud ID for API calls
- `jira.siteUrl` - Atlassian site URL
- `jira.projectKey` - Project key for issue creation
- `notion.enabled` - Whether Notion sync is enabled

### 0.2 Load Notion Config

```bash
cat ~/.claude/notion.json 2>/dev/null
```

If `todo.id` not found:
```
✗ Notion TODO database not configured.
Run /workflow:init to set up Notion TODO sync.
```

Extract:
- `todo.id` - TODO database data source ID

## Step 1: Query Notion TODO Items

Fetch TODO database to find items without Jira Link:

```
mcp__notion__notion-fetch:
  id: {todo.id}
```

Parse the response and filter items where:
- `Jira Link` field is empty or doesn't contain a valid issue key

If no items found:
```
✓ All TODO items already have Jira links.
Nothing to sync.
```

## Step 2: Show Items to Sync

Display list of TODO items:

```
Found {n} items without Jira Link:

1. {title1}
   ├─ Epic: {epic_field}
   ├─ Priority: {priority}
   └─ Due: {due_date}

2. {title2}
   ├─ Epic: (none)
   ├─ Priority: {priority}
   └─ Due: (none)

...
```

**Ask user (AskUserQuestion):**
"Sync these items to Jira?"

| Option | Description |
|--------|-------------|
| All | Sync all {n} items |
| Cancel | Exit without syncing |

## Step 3: Get Current User

```
mcp__atlassian__atlassianUserInfo
```

Extract `{current_user_account_id}` for auto-assignment.

## Step 4: Process Each Item

For each selected item:

### 4.1 Parse Epic Field

Parse the Epic field to determine action:

```
Epic field value → Action
─────────────────────────────────────────────
"CP-1: Login feature"  → Extract key "CP-1", use as parent
"CP-1"                 → Use as parent directly
"Login feature"        → Create new Epic, use as parent
""                     → No parent (standalone)
```

**Regex for key extraction:** `/^([A-Z]+-\d+)/` or `/([A-Z]+-\d+):/`

### 4.2 Create Epic (if needed)

If Epic has name but no key:

```
mcp__atlassian__createJiraIssue:
  cloudId: {jira.cloudId}
  projectKey: {jira.projectKey}
  issueTypeName: "Epic"  # Use localized name if needed (에픽)
  summary: {epic_name}
  description: ""
  assignee_account_id: {current_user_account_id}
```

Store new Epic key for use as parent.

### 4.3 Create Task Issue

```
mcp__atlassian__createJiraIssue:
  cloudId: {jira.cloudId}
  projectKey: {jira.projectKey}
  issueTypeName: "Task"  # Use localized name if needed (작업)
  summary: {notion_title}
  description: ""
  parent: {epic_key}  # Only if Epic exists
  assignee_account_id: {current_user_account_id}
```

### 4.4 Set Due Date (if exists)

If Notion item has Due Date:

```
mcp__atlassian__editJiraIssue:
  cloudId: {jira.cloudId}
  issueIdOrKey: {created_issue_key}
  fields: { "duedate": "{due_date}" }
```

### 4.5 Update Notion TODO Item

Update the Notion page with Jira Link (and Epic key if created):

```
mcp__notion__notion-update-page:
  data: {
    "page_id": "{notion_page_id}",
    "command": "update_properties",
    "properties": {
      "Jira Link": "[{issueKey}](https://{jira.siteUrl}/browse/{issueKey})",
      "Epic": "{epicKey}: {epicName}"  # Only if Epic was created
    }
  }
```

## Step 5: Summary

### On Success:

```
✓ Synced {n} items to Jira

{if epics_created}
Created Epics:
├─ {epic_key1}: {epic_name1}
└─ {epic_key2}: {epic_name2}
{/if}

Issues:
├─ {issue_key1}: {title1} (under {epic_key})
├─ {issue_key2}: {title2} (under {epic_key})
└─ {issue_key3}: {title3} (standalone)

Next steps:
- /jira:start {first_issue_key} - Start working on an issue
- /jira:view {first_issue_key} - View issue details
```

### On Partial Failure:

```
⚠ Synced {success_count}/{total_count} items

Succeeded:
├─ {issue_key1}: {title1}
└─ {issue_key2}: {title2}

Failed:
└─ {title3}: {error_message}
```

## Constraints

1. **Only sync items without Jira Link**: Prevents duplicates
2. **Default issue type: Task**: Simplest and most common
3. **Auto-assign to current user**: Consistent with /jira:create
4. **Epic handling**:
   - Key found → use existing Epic
   - Name only → create new Epic
   - Empty → standalone issue
5. **Status not synced**: New issues start as "To Do"
