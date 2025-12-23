---
description: Start working on a Jira issue (creates branch, updates status)
---

# Start Jira Issue

Start working on a Jira issue by creating a branch, updating status, and syncing Notion.

## Usage

```
/jira:start [issue-key]
```

If no issue key provided, shows list of available issues.

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
- `jira.projectKey` - Project key
- `jira.includeInBranch` - Include issue key in branch
- `notion.enabled` - Whether to sync to Notion

### 0.2 Load Notion Config (if notion.enabled)

```bash
cat ~/.claude/notion.json 2>/dev/null
```

Extract:
- `todo.id` - TODO database ID (user-level config)

## Step 1: Select Issue (if no argument)

If issue key not provided as argument:

Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
jql: "project = {projectKey} AND assignee = currentUser() AND status = 'To Do' ORDER BY priority DESC"
fields: ["summary", "status", "issuetype", "priority"]
maxResults: 10
```

Display issues using **Cascading Selection** (4-option limit):

**Round 1:**
| Option | Description |
|--------|-------------|
| {issue1.key} | {issue1.summary} |
| {issue2.key} | {issue2.summary} |
| {issue3.key} | {issue3.summary} |
| More... | Show more issues |

**Round 2 (if More...):**
| Option | Description |
|--------|-------------|
| {issue4.key} | {issue4.summary} |
| {issue5.key} | {issue5.summary} |
| {issue6.key} | {issue6.summary} |
| Enter key | Enter issue key manually |

## Step 2: Get Issue Details

Call `mcp__atlassian__getJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {selected_issue_key}
fields: ["summary", "status", "issuetype", "priority", "description"]
```

Verify issue exists and extract:
- `issueKey` - e.g., CP-1
- `summary` - Issue title
- `issueType` - Bug, Task, Story, Epic
- `status` - Current status

## Step 3: Transition Issue to In Progress + Set Start Date

### 3.1: Transition to In Progress

Get available transitions:
Call `mcp__atlassian__getTransitionsForJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
```

Find transition to "In Progress" status and execute:
Call `mcp__atlassian__transitionJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
transition: {"id": "{transition_id}"}
```

### 3.2: Set Jira Start Date

1. Get current date:
   ```bash
   date +%Y-%m-%d
   ```
   Store as `{today_date}`.

2. Set the start date field in Jira:
   Call `mcp__atlassian__editJiraIssue`:
   ```
   cloudId: {jira.cloudId}
   issueIdOrKey: {issueKey}
   fields: {
     "customfield_10015": "{today_date}"
   }
   ```

**Note:** Start Date field name may vary by project:
- Common names: `customfield_10015`, `customfield_10014`, `Start date`
- If field not found, skip with warning (non-critical)

## Step 4: Create Branch (via /git:branch)

Map issue type to branch type:
- Bug → `bugfix`
- Story → `feature`
- Task → `feature`
- Epic → `feature`

Convert summary to kebab-case description:
- "Add user authentication" → "add-user-authentication"
- Remove special characters, lowercase, replace spaces with hyphens
- Truncate to 50 characters if needed

**Internally call `/git:branch`** with parameters:
- type: {mapped_branch_type}
- issueKey: {issueKey}
- description: {kebab_case_summary}

This creates branch: `{type}/{issueKey}-{description}`
Example: `feature/CP-1-add-user-authentication`

## Step 5: Update Notion TODO (if enabled)

If `notion.enabled` is true:

1. Search for existing TODO item by Jira link:
   Call `mcp__notion__notion-search`:
   ```
   query: "{issueKey}"
   filter: {"property": "object", "value": "page"}
   ```

2. If found, update status and start date:
   Call `mcp__notion__notion-update-page`:
   ```
   data: {
     "page_id": "{found_page_id}",
     "command": "update_properties",
     "properties": {
       "Status": "In Progress",
       "date:Start Date:start": "{today_date}",
       "date:Start Date:is_datetime": 0
     }
   }
   ```

   Where `{today_date}` is today's date in `YYYY-MM-DD` format.

## Output

### On Success:
```
✓ Issue started: {issueKey}

Summary: {summary}
Type: {issueType}
Status: To Do → In Progress
Start Date: {today_date}

Branch: {branch_name}
├─ Created via /git:branch
└─ Checked out automatically

{if notion synced}
✓ Notion TODO updated
  ├─ Status: Todo → In Progress
  └─ Start Date: {today_date}
{/if}

Next steps:
- Make your changes
- /git:commit - Commit with [{issueKey}] in message
- /jira:done - Complete the issue and create PR
```

### On Issue Not Found:
```
✗ Issue not found: {issueKey}

Check the issue key and try again.
Use /jira:backlog to see available issues.
```

### On Transition Failed:
```
⚠ Issue started with warnings

Branch created: {branch_name}
Status change failed: {error}

Issue may already be In Progress or transition not available.
```

### On Branch Creation Failed:
```
✗ Failed to start issue: {error}

Possible causes:
- Uncommitted changes in current branch
- Branch already exists
- Invalid branch name

Fix the issue and try again.
```

## Constraints

1. **Check configuration first**: Ensure workflow.json exists
2. **Validate issue exists**: Verify issue before creating branch
3. **Use /git:branch internally**: Don't run git checkout directly
4. **Handle partial failures**: Branch may succeed even if status change fails
5. **Keep branch names clean**: Sanitize summary for valid branch names
