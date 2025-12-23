---
description: Create a new Jira issue
---

# Create Jira Issue

Create a new issue in the configured Jira project with optional Notion TODO sync.

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
- `jira.projectKey` - Project key for issue creation
- `notion.enabled` - Whether to sync to Notion

### 0.2 Load Notion Config (if notion.enabled)

```bash
cat ~/.claude/notion.json 2>/dev/null
```

Extract:
- `todo.id` - TODO database ID (user-level config)

## Step 1: Select Issue Type

**Ask user (AskUserQuestion):**
"Select issue type:"

| Option | Description |
|--------|-------------|
| Task | Small individual work item |
| Bug | Problem or error to fix |
| Story | User story or feature |
| Epic | Collection of related issues |

## Step 2: Enter Summary

**Ask user (AskUserQuestion):**
"Enter issue summary (title):"

User enters the summary text.

## Step 3: Enter Description (Optional)

**Ask user (AskUserQuestion):**
"Would you like to add a description?"

| Option | Description |
|--------|-------------|
| Yes | Add detailed description |
| No | Create without description |

If Yes, prompt for description text.

## Step 4: Select Priority

**Ask user (AskUserQuestion):**
"Select priority:"

| Option | Description |
|--------|-------------|
| High | Urgent, needs immediate attention |
| Medium | Normal priority (Recommended) |
| Low | Can be addressed later |

## Step 5: Create Jira Issue

Call `mcp__atlassian__createJiraIssue`:
```
cloudId: {jira.cloudId}
projectKey: {jira.projectKey}
issueTypeName: {selected_type}
summary: {entered_summary}
description: {entered_description} (if provided)
additional_fields: {
  "priority": {"name": "{selected_priority}"}
}
```

## Step 6: Sync to Notion TODO (if enabled)

If `notion.enabled` is true (from workflow.json) and `todo.id` exists (from ~/.claude/notion.json):

1. Get current project name:
   ```bash
   basename $(pwd)
   ```

2. Create Notion TODO item via `mcp__notion__notion-create-page`:
   ```
   database_id: {todo.id}  # from ~/.claude/notion.json
   properties: {
     "Title": {
       "title": [{"text": {"content": "{summary}"}}]
     },
     "Status": {
       "status": {"name": "Todo"}
     },
     "Project": {
       "select": {"name": "{project_name}"}
     },
     "Jira Link": {
       "url": "https://{cloudId}/browse/{issueKey}"
     },
     "Priority": {
       "select": {"name": "{priority}"}
     }
   }
   ```

**Note:** If `todo.id` not found in user config, skip Notion sync with message:
```
⚠️ Notion TODO database not configured.
Run /workflow:init to set up Notion TODO sync.
```

## Step 7: Start Immediately?

**Ask user (AskUserQuestion):**
"Would you like to start working on this issue now?"

| Option | Description |
|--------|-------------|
| Yes | Run /jira:start {issueKey} |
| No | Just create the issue |

If Yes, execute `/jira:start {issueKey}`.

## Output

### On Success:
```
✓ Issue created: {issueKey}

Summary: {summary}
Type: {type}
Priority: {priority}
URL: https://{cloudId}/browse/{issueKey}

{if notion synced}
✓ Notion TODO item created

{/if}
Next steps:
- /jira:start {issueKey} - Start working on this issue
- /jira:view {issueKey} - View issue details
```

### On Failure:
```
✗ Failed to create issue: {error message}
```

## Constraints

1. **Check configuration first**: Ensure workflow.json exists and Jira is enabled
2. **Validate summary**: Summary is required and should not be empty
3. **Handle Notion sync gracefully**: If Notion sync fails, still show Jira success
4. **Use project key from config**: Don't ask user for project
