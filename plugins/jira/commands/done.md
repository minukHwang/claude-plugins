---
description: Complete a Jira issue (creates PR, updates status, syncs Notion)
---

# Complete Jira Issue

Complete a Jira issue by creating a PR, updating status, and syncing Notion.

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
- `notion.enabled` - Whether to sync to Notion

### 0.2 Load Notion Config (if notion.enabled)

```bash
cat ~/.claude/notion.json 2>/dev/null
```

Extract:
- `todo.id` - TODO database ID (user-level config)

## Step 1: Extract Issue Key from Branch

```bash
git branch --show-current
```

Extract issue key using pattern: `[A-Z]+-[0-9]+`
Example: `feature/CP-1-add-auth` → `CP-1`

If no issue key found:
```
⚠ No Jira issue key found in branch name.

Current branch: {branch_name}
Expected format: feature/CP-1-description

Continue without Jira integration?
```

**Ask user (AskUserQuestion):**
| Option | Description |
|--------|-------------|
| Continue | Create PR without Jira link |
| Cancel | Abort operation |
| Enter key | Manually enter issue key |

## Step 2: Get Issue Details

Call `mcp__atlassian__getJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
fields: ["summary", "status", "issuetype"]
```

Display current state:
```
Issue: {issueKey}
Summary: {summary}
Current Status: {status}
```

## Step 3: Check for Commits

```bash
git log origin/main..HEAD --oneline
```

If no commits:
```
⚠ No commits found on this branch.

Make some changes and commit before completing the issue.
Use /git:commit to create a commit.
```

## Step 4: Create Pull Request (via /git:pr)

**Internally call `/git:pr`** which will:
- Create PR with standard format
- Include Jira link in PR body (Phase 3 modification)

The PR body will include:
```markdown
## Jira Issue
[{issueKey}](https://{cloudId}/browse/{issueKey}) - {summary}
```

Capture PR URL from the result.

## Step 5: Select New Status

**Ask user (AskUserQuestion):**
"Update Jira issue status to:"

| Option | Description |
|--------|-------------|
| In Review | PR is under review |
| Done | Issue is completed |
| Skip | Don't change status |

### If In Review or Done

Get available transitions:
Call `mcp__atlassian__getTransitionsForJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
```

Find and execute the selected transition:
Call `mcp__atlassian__transitionJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
transition: {"id": "{transition_id}"}
```

## Step 6: Update Notion TODO (if enabled)

If `notion.enabled` is true:

1. Search for TODO item by Jira link:
   Call `mcp__notion__notion-search`:
   ```
   query: "{issueKey}"
   ```

2. Get the latest commit hash:
   ```bash
   git log -1 --format="%H"
   ```

3. Get repository URL:
   ```bash
   git remote get-url origin
   ```

4. Update Notion page:
   Call `mcp__notion__notion-update-page`:
   ```
   data: {
     "page_id": "{found_page_id}",
     "command": "update_properties",
     "properties": {
       "Status": "{selected_status}",
       "Commit": "{repo_url}/commit/{commit_hash}",
       "PR": "{pr_url}"
     }
   }
   ```

## Output

### On Success:
```
✓ Issue completed: {issueKey}

Summary: {summary}
Status: {old_status} → {new_status}

Pull Request:
└─ {pr_url}

Jira Issue:
└─ https://{cloudId}/browse/{issueKey}

{if notion synced}
Notion TODO Updated:
├─ Status: {new_status}
├─ Commit: {commit_url}
└─ PR: {pr_url}
{/if}

Next steps:
- Review and merge PR
- Close issue after merge
```

### On PR Creation Failed:
```
✗ Failed to create PR: {error}

Possible causes:
- No commits to push
- PR already exists for this branch
- Push permission denied

Fix the issue and try again.
```

### On Status Change Failed:
```
⚠ Issue completed with warnings

PR created: {pr_url}
Status change failed: {error}

Please update Jira status manually.
```

## Constraints

1. **Check for commits first**: Verify there are commits before creating PR
2. **Use /git:pr internally**: Don't create PR directly
3. **Handle partial failures**: PR may succeed even if status change fails
4. **Extract issue key from branch**: Use regex pattern matching
5. **Update all tracking systems**: Jira + Notion if configured
