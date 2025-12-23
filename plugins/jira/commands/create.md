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

## Step 1.5: Select Parent Epic (if not Epic type)

If selected type is NOT Epic (에픽):

1. Query existing Epics in project:
   ```
   mcp__atlassian__searchJiraIssuesUsingJql:
     cloudId: {jira.cloudId}
     jql: "project = {jira.projectKey} AND type = Epic ORDER BY created DESC"
     maxResults: 10
     fields: ["summary"]
   ```

2. If Epics found, ask user (Cascading Selection):

   **Round 1 (first 3 + None/More):**

   **Ask user (AskUserQuestion):**
   "상위 Epic을 선택하세요:"

   | Option | Description |
   |--------|-------------|
   | {epic1.key} | {epic1.summary} |
   | {epic2.key} | {epic2.summary} |
   | {epic3.key} | {epic3.summary} |
   | More.../None | Show more or create standalone |

   **Round 2 (if More...):**
   | Option | Description |
   |--------|-------------|
   | {epic4.key} | {epic4.summary} |
   | {epic5.key} | {epic5.summary} |
   | {epic6.key} | {epic6.summary} |
   | None | Create as standalone issue |

3. If no Epics found:
   - Skip this step
   - Display: "No Epics found in project. Creating standalone issue."

4. Store selected Epic info:
   - `{selected_epic_key}` - Epic issue key (e.g., CP-1)
   - `{selected_epic_summary}` - Epic summary for Notion

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

## Step 4.5: Set Due Date (Optional)

**Ask user (AskUserQuestion):**
"Set due date?"

| Option | Description |
|--------|-------------|
| Yes | Enter due date |
| No | Create without due date |

If Yes:
1. Get current date:
   ```bash
   date +%Y-%m-%d
   ```

2. Calculate date options based on current date:
   - 1 week later: `{current_date} + 7 days`
   - 2 weeks later: `{current_date} + 14 days`
   - End of month: Last day of current month
   - Custom: User enters manually

3. Prompt user for date selection (YYYY-MM-DD format)
4. Store as `{due_date}`

## Step 5: Create Jira Issue

### 5.1: Get Current User Account ID

```
mcp__atlassian__atlassianUserInfo
```

Extract `{current_user_account_id}` from response.

### 5.2: Create Issue

Call `mcp__atlassian__createJiraIssue`:
```
cloudId: {jira.cloudId}
projectKey: {jira.projectKey}
issueTypeName: {selected_type}  # Use localized name from Step 1
summary: {entered_summary}
description: {entered_description}  # Required, use "" if empty
parent: {selected_epic_key}  # Only if Epic selected in Step 1.5
assignee_account_id: {current_user_account_id}  # Auto-assign to self
```

### 5.3: Set Due Date (if provided)

If `{due_date}` was set in Step 4.5:

```
mcp__atlassian__editJiraIssue:
  cloudId: {jira.cloudId}
  issueIdOrKey: {created_issue_key}
  fields: { "duedate": "{due_date}" }
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
       "Epic": "{selected_epic_key}: {selected_epic_summary}",  # Only if Epic selected
       "Status": "Todo",
       "Project": "[{project}](https://{host}/{namespace}/{project})",
       "Jira Link": "[{issueKey}](https://{jira.siteUrl}/browse/{issueKey})",
       "Priority": "{priority}"
     }
   }]
   ```

   **Note:** If no Epic selected, omit the "Epic" field or set to empty string.

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
Parent: {epic_key} ({epic_summary})  # if Epic selected
Priority: {priority}
Due Date: {due_date}  # if set
Assignee: {current_user_name}
URL: https://{jira.siteUrl}/browse/{issueKey}

{if notion synced}
✓ Notion TODO item created
  └── Epic: {epic_key}: {epic_summary}
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
