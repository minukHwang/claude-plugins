---
description: View Jira issue details
---

# View Jira Issue

Display detailed information about a Jira issue.

## Usage

```
/jira:view [issue-key]
```

If no issue key provided, extracts from current branch or prompts for selection.

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
- `jira.projectKey` - Project key

## Step 1: Determine Issue Key

### If issue key provided as argument
Use the provided issue key.

### If no argument
Try to extract from current branch:
```bash
git branch --show-current
```

Extract issue key using pattern: `[A-Z]+-[0-9]+`

If no issue key found in branch:

**Ask user (AskUserQuestion):**
"Enter issue key or select from recent issues:"

Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
jql: "project = {projectKey} ORDER BY updated DESC"
maxResults: 6
```

Display using Cascading Selection:

| Option | Description |
|--------|-------------|
| {issue1.key} | {issue1.summary} |
| {issue2.key} | {issue2.summary} |
| {issue3.key} | {issue3.summary} |
| Enter key | Enter issue key manually |

## Step 2: Get Issue Details

Call `mcp__atlassian__getJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
fields: ["summary", "description", "status", "issuetype", "priority", "assignee", "reporter", "created", "updated", "labels", "components"]
```

## Step 3: Get Comments (Optional)

Check if issue has comments and retrieve recent ones.

## Step 4: Get Related Links

Call `mcp__atlassian__getJiraIssueRemoteIssueLinks`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
```

## Output

### On Success:
```
{issueKey}: {summary}
{'=' * width}

Type: {issuetype}
Status: {status}
Priority: {priority}

Assignee: {assignee.displayName}
Reporter: {reporter.displayName}

Created: {created}
Updated: {updated}

Labels: {labels.join(', ')}
Components: {components.join(', ')}

Description:
{description}

{if has_links}
Related Links:
{for each link}
- {link.title}: {link.url}
{/for}
{/if}

{if has_comments}
Recent Comments:
{for each comment}
---
{comment.author} ({comment.created}):
{comment.body}
{/for}
{/if}

URL: https://{cloudId}/browse/{issueKey}
```

### On Issue Not Found:
```
✗ Issue not found: {issueKey}

Check the issue key and try again.
Use /jira:backlog to see available issues.
```

## Step 5: Quick Actions

**Ask user (AskUserQuestion):**
"Would you like to take action?"

| Option | Description |
|--------|-------------|
| Start Issue | Run /jira:start {issueKey} |
| Add Comment | Add a comment to this issue |
| Open in Browser | Open Jira in browser |
| Done | Exit |

### If Add Comment

**Ask user (AskUserQuestion):**
"Enter your comment:"

User enters comment text.

Call `mcp__atlassian__addCommentToJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
commentBody: {entered_comment}
```

### If Open in Browser

```bash
open "https://{cloudId}/browse/{issueKey}"
```

## Constraints

1. **Check configuration first**: Ensure workflow.json exists
2. **Handle missing fields gracefully**: Not all fields may be present
3. **Format description properly**: May contain markdown
4. **Limit comment display**: Show only recent comments
5. **Provide actionable next steps**: Offer relevant actions
