---
description: View project backlog issues
---

# View Backlog

Display backlog issues from the configured Jira project.

## Step 0: Load Workflow Configuration

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
- `jira.projectKey` - Project key for queries

## Step 1: Select View Type

**Ask user (AskUserQuestion):**
"What would you like to view?"

| Option | Description |
|--------|-------------|
| Backlog | Issues not in any sprint |
| My Issues | Issues assigned to me |
| All Open | All open issues in project |

## Step 2: Query Issues

### If Backlog

Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
jql: "project = {projectKey} AND sprint is EMPTY AND status != Done ORDER BY priority DESC, created DESC"
fields: ["summary", "status", "issuetype", "priority", "assignee", "created"]
maxResults: 25
```

### If My Issues

Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
jql: "project = {projectKey} AND assignee = currentUser() AND status != Done ORDER BY priority DESC, updated DESC"
fields: ["summary", "status", "issuetype", "priority", "created"]
maxResults: 25
```

### If All Open

Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
jql: "project = {projectKey} AND status != Done ORDER BY priority DESC, updated DESC"
fields: ["summary", "status", "issuetype", "priority", "assignee", "created"]
maxResults: 25
```

## Step 3: Display Results

Format as table:

```
{View Type} - {projectKey}
{'=' * width}

| Key | Type | Summary | Status | Priority |
|-----|------|---------|--------|----------|
| CP-1 | Task | Add user login | To Do | High |
| CP-2 | Bug | Fix header alignment | In Progress | Medium |
| CP-3 | Story | Dashboard redesign | To Do | Low |

Total: {count} issues
```

## Step 4: Quick Actions

**Ask user (AskUserQuestion):**
"Would you like to take action on an issue?"

| Option | Description |
|--------|-------------|
| Start Issue | Run /jira:start on an issue |
| View Details | Run /jira:view on an issue |
| Create New | Run /jira:create |
| Done | Exit |

### If Start Issue or View Details

**Ask user (AskUserQuestion):**
"Enter issue key (e.g., CP-1):"

Display issue keys from the list as options (Cascading Selection if > 3):

| Option | Description |
|--------|-------------|
| {issue1.key} | {issue1.summary} |
| {issue2.key} | {issue2.summary} |
| {issue3.key} | {issue3.summary} |
| More... | Show more issues |

Then execute the corresponding command.

## Output

### On Success:
```
Backlog - CP
============

| Key | Type | Summary | Status | Priority |
|-----|------|---------|--------|----------|
| CP-1 | Task | Add user login | To Do | High |
| CP-2 | Bug | Fix header alignment | In Progress | Medium |

Total: 2 issues

Use /jira:start <key> to start working on an issue.
```

### On No Results:
```
No issues found in backlog.

Use /jira:create to create a new issue.
```

### On Error:
```
✗ Failed to fetch issues: {error message}
```

## Constraints

1. **Check configuration first**: Ensure workflow.json exists and Jira is enabled
2. **Limit results**: Default to 25 issues to avoid overwhelming output
3. **Sort by priority**: Show high priority issues first
4. **Show actionable info**: Include status and type for quick decisions
