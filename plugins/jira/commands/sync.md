---
description: Sync Notion TODO items to Jira (creates missing issues)
---

# Bidirectional Sync: Jira ↔ Notion TODO

Sync TODO items between Jira and Notion bidirectionally.

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

## Step 1: Select Sync Scope

**Ask user (AskUserQuestion):**
"Select sync scope:"

| Option | Description |
|--------|-------------|
| Current project | Only {projectKey} from workflow.json |
| All projects | All Jira-linked TODOs across projects |

Store selection as `{sync_scope}`.

## Step 2: Query Both Systems

### 2.1 Query Jira Issues

Get current date for query:
```bash
date +%Y-%m-%d
```

Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
cloudId: {jira.cloudId}
jql: "project = {projectKey} AND assignee = currentUser() ORDER BY updated DESC"
  # or "assignee = currentUser() ORDER BY updated DESC" for all projects
fields: ["summary", "status", "issuetype", "priority", "duedate", "customfield_10015", "updated"]
maxResults: 100
```

### 2.2 Query Notion TODOs

Fetch TODO database:
```
mcp__notion__notion-fetch:
  id: {todo.id}
```

Filter items where:
- `Type` != "Todo" (exclude personal items)
- If `sync_scope` is "Current project": `Project` matches current repo

Parse each item's:
- Title
- Status
- Priority
- Type
- Period (start/end dates)
- Jira Link (extract issue key if present)
- last_edited_time

## Step 3: Match Items

Create three categories:

### 3.1 Jira Only (New to Notion)
Issues in Jira that don't have matching Notion TODO (by issue key in Jira Link).

### 3.2 Notion Only (New to Jira)
TODO items without Jira Link AND Type != "Todo".

### 3.3 Both Exist (Need Sync)
Items that exist in both systems (matched by Jira issue key).

## Step 4: Show Sync Preview

```
📊 Sync Preview

Jira → Notion (new items):
├─ CP-1: Add authentication
├─ CP-3: Fix login bug
└─ (2 items)

Notion → Jira (new items):
├─ Implement dashboard
├─ Add user settings
└─ (2 items)

Updates needed (by latest edit):
├─ CP-2: Jira is newer → update Notion
├─ CP-5: Notion is newer → update Jira
├─ CP-6: One side Done → sync Done
└─ (3 items)

Excluded:
└─ 5 personal items (Type = Todo)
```

**Ask user (AskUserQuestion):**
"Proceed with sync?"

| Option | Description |
|--------|-------------|
| Yes | Sync all {n} items |
| Cancel | Exit without syncing |

## Step 5: Process Sync

### 5.1 Jira → Notion (Create new TODO)

For each Jira-only issue:

1. Get current user for assignment:
   ```
   mcp__atlassian__atlassianUserInfo
   ```

2. Get git remote info:
   ```bash
   git remote get-url origin
   ```

3. Map Jira fields to Notion:
   | Jira Field | Notion Field |
   |------------|--------------|
   | summary | Title |
   | issuetype | Type (Epic/Story/Task/Bug) |
   | status | Status (Todo/In Progress/Done) |
   | priority | Priority |
   | customfield_10015 | Period:start |
   | duedate | Period:end |

4. Create Notion TODO:
   ```
   mcp__notion__notion-create-pages:
     parent: {"type": "data_source_id", "data_source_id": "{todo.id}"}
     pages: [{
       "properties": {
         "Title": "{summary}",
         "Type": "{issue_type}",
         "Status": "{mapped_status}",
         "Priority": "{priority}",
         "Project": "[{project}](https://{host}/{namespace}/{project})",
         "Jira Link": "[{issueKey}](https://{jira.siteUrl}/browse/{issueKey})",
         "date:Period:start": "{start_date}",
         "date:Period:end": "{duedate}",
         "date:Period:is_datetime": 0
       }
     }]
   ```

### 5.2 Notion → Jira (Create new issue)

For each Notion-only item (Type != Todo):

1. Create Jira issue:
   ```
   mcp__atlassian__createJiraIssue:
     cloudId: {jira.cloudId}
     projectKey: {jira.projectKey}
     issueTypeName: "{notion_type}"  # Task if Type not set
     summary: {notion_title}
     description: ""
     assignee_account_id: {current_user_account_id}
   ```

2. Set Due Date and Start Date:
   ```
   mcp__atlassian__editJiraIssue:
     cloudId: {jira.cloudId}
     issueIdOrKey: {created_issue_key}
     fields: {
       "duedate": "{due_date}",
       "customfield_10015": "{start_date}"
     }
   ```

3. Update Notion with Jira Link:
   ```
   mcp__notion__notion-update-page:
     data: {
       "page_id": "{notion_page_id}",
       "command": "update_properties",
       "properties": {
         "Jira Link": "[{issueKey}](https://{jira.siteUrl}/browse/{issueKey})"
       }
     }
   ```

### 5.3 Both Exist (Sync Updates)

For each matched pair:

1. Compare timestamps:
   - Jira: `updated` field
   - Notion: `last_edited_time`

2. Determine sync direction:
   - Jira newer → Update Notion
   - Notion newer → Update Jira
   - Same → Skip

3. **Done Status Propagation:**
   If either side is Done → sync Done to other side.

4. Sync fields based on winner:
   | Field | Jira → Notion | Notion → Jira |
   |-------|---------------|---------------|
   | Status | status → Status | Status → transition |
   | Priority | priority → Priority | Priority → priority |
   | Period:start | customfield_10015 → Period:start | Period:start → customfield_10015 |
   | Period:end | duedate → Period:end | Period:end → duedate |

   **Note:** Summary is NOT synced to prevent accidental overwrites.

5. Update loser side:

   **Update Notion:**
   ```
   mcp__notion__notion-update-page:
     data: {
       "page_id": "{notion_page_id}",
       "command": "update_properties",
       "properties": {
         "Status": "{jira_status}",
         "Priority": "{jira_priority}",
         "date:Period:start": "{jira_start_date}",
         "date:Period:end": "{jira_duedate}",
         "date:Period:is_datetime": 0
       }
     }
   ```

   **Update Jira:**
   First get transitions:
   ```
   mcp__atlassian__getTransitionsForJiraIssue:
     cloudId: {jira.cloudId}
     issueIdOrKey: {issueKey}
   ```

   Then transition and edit:
   ```
   mcp__atlassian__transitionJiraIssue:
     cloudId: {jira.cloudId}
     issueIdOrKey: {issueKey}
     transition: {"id": "{transition_id}"}
   ```

   ```
   mcp__atlassian__editJiraIssue:
     cloudId: {jira.cloudId}
     issueIdOrKey: {issueKey}
     fields: {
       "priority": {"name": "{notion_priority}"},
       "customfield_10015": "{period_start}",
       "duedate": "{period_end}"
     }
   ```

## Step 6: Summary

### On Success:

```
✓ Sync completed

Jira → Notion:
├─ Created: {n1} items
└─ Updated: {n2} items

Notion → Jira:
├─ Created: {n3} issues
└─ Updated: {n4} issues

Done propagated: {n5} items

Excluded:
└─ {n6} personal items (Type = Todo)

Next steps:
- /jira:start {first_issue_key} - Start working on an issue
- /jira:view {first_issue_key} - View issue details
```

### On Partial Failure:

```
⚠ Sync completed with errors

Succeeded: {success_count}/{total_count}

Failed:
├─ {item1}: {error_message}
└─ {item2}: {error_message}
```

## Status Mapping

| Jira Status | Notion Status |
|-------------|---------------|
| To Do | Todo |
| In Progress | In Progress |
| In Review | In Review |
| Done, Closed, Resolved | Done |

## Constraints

1. **Exclude personal items**: Type = "Todo" is never synced to Jira
2. **Project scope**: Optionally limit to current project only
3. **Conflict resolution**: Latest edit wins
4. **Done propagation**: Done status always syncs to other side
5. **No summary sync**: Prevents accidental title overwrites
6. **Auto-assign**: New Jira issues assigned to current user
