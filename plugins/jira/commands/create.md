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
- `jira.siteUrl` - Atlassian site URL (e.g., yoursite.atlassian.net)
- `jira.projectKey` - Project key for issue creation
- `notion.enabled` - Whether to sync to Notion

### 0.2 Load Notion Config (if notion.enabled)

```bash
cat ~/.claude/notion.json 2>/dev/null
```

Extract:
- `todo.id` - TODO database ID (user-level config)

## Step 1: Get Issue Types from Project

First, fetch available issue types:
```
mcp__atlassian__getJiraProjectIssueTypesMetadata:
  cloudId: {jira.cloudId}
  projectIdOrKey: {jira.projectKey}
```

**Note:** Issue type names may be localized (e.g., Korean: 작업, 버그, 스토리, 에픽).
Use the `name` field from the API response, not hardcoded English names.

**Ask user (AskUserQuestion):**
"Select issue type:"

Present options from API response (common types):
| Option | Description |
|--------|-------------|
| Task (작업) | Small individual work item |
| Bug (버그) | Problem or error to fix |
| Story (스토리) | User story or feature |
| Epic (에픽) | Collection of related issues |

## Step 2: Enter Summary

**Ask user (AskUserQuestion):**
"Enter issue summary (title):"

User enters the summary text.

## Step 3: Enter Description

**Ask user (AskUserQuestion):**
"Enter issue description (or leave empty):"

**Note:** Description field is required by Jira API. If user skips, use empty string "".

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
issueTypeName: {selected_type}  # Use localized name from Step 1
summary: {entered_summary}
description: {entered_description}  # Required, use "" if empty
```

**Note:** Priority via additional_fields may cause "Input data should be a String" error.
Set priority separately via `mcp__atlassian__editJiraIssue` if needed.

## Step 6: Sync to Notion TODO (if enabled)

If `notion.enabled` is true (from workflow.json) and `todo.id` exists (from ~/.claude/notion.json):

1. Get git remote info:
   ```bash
   git remote get-url origin
   ```

   Parse the URL to extract:
   - `{host}`: github.com or gitlab.com
   - `{namespace}`: owner/org name
   - `{project}`: repository name

   Format project as markdown link: `[{project}](https://{host}/{namespace}/{project})`

2. Create Notion TODO item via `mcp__notion__notion-create-pages`:
   ```
   parent: {"type": "data_source_id", "data_source_id": "{todo.id}"}
   pages: [{
     "properties": {
       "Title": "{summary}",
       "Status": "Todo",
       "Project": "[{project}](https://{host}/{namespace}/{project})",
       "Jira Link": "https://{jira.siteUrl}/browse/{issueKey}",
       "Priority": "{priority}"
     }
   }]
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
