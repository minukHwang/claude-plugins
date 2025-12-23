---
description: Create a new git branch with type/description format and checkout
---

# Create Git Branch

Create a new branch following the naming convention and automatically checkout.

## Branch Naming Convention

**Standard format:**
```
<type>/<description>
```

**With Jira integration:**
```
<type>/<issueKey>-<description>
```

**Team/Collaborative project:**
```
<accountName>/<type>/<description>
```

## Step 0: Load Workflow Configuration (Optional)

```bash
cat .claude/workflow.json 2>/dev/null
```

If exists, extract:
- `jira.enabled` - Whether Jira integration is active
- `jira.cloudId` - Atlassian cloud ID
- `jira.projectKey` - Current project key
- `jira.includeInBranch` - Include issue key in branch names
- `notion.enabled` - Whether Notion integration is active
- `notion.databases.todo.id` - TODO database ID

## Step 1: Ask for Branch Type (Cascading Selection)

Use cascading selection to show all branch types:

### Round 1 - Common Types

**Ask user (AskUserQuestion):**
"Select the branch type:

Available: feature, bugfix, hotfix, release, docs, refactor, style, test, chore"

| Option | Description |
|--------|-------------|
| feature | Adding a new feature or capability |
| bugfix | Fixing a bug or issue |
| hotfix | Urgent fix for production |
| More types | Show more options... |

### Round 2 - If "More types" selected

**Ask user (AskUserQuestion):**
"Select the branch type:"

| Option | Description |
|--------|-------------|
| release | Release preparation |
| docs | Documentation changes |
| refactor | Code restructuring |
| More types | Show remaining options... |

### Round 3 - If "More types" selected again

**Ask user (AskUserQuestion):**
"Select the branch type:"

| Option | Description |
|--------|-------------|
| style | Code style/formatting changes |
| test | Testing related changes |
| chore | Routine maintenance tasks |

User can also enter custom type via "Other" option (kebab-case).

## Step 1.5: Link Jira Issue (if jira.enabled && jira.includeInBranch)

If workflow.json has `jira.enabled: true` and `jira.includeInBranch: true`:

**Ask user (AskUserQuestion):**
"Would you like to link a Jira issue?"

| Option | Description |
|--------|-------------|
| Select issue | Choose from my assigned issues |
| Enter issue key | Enter issue key manually (e.g., CP-123) |
| No link | Create branch without issue |

### If "Select issue"

Query assigned issues:
Call `mcp__atlassian__searchJiraIssuesUsingJql`:
```
jql: "project = {projectKey} AND assignee = currentUser() AND status = 'To Do' ORDER BY priority DESC"
fields: ["summary", "status", "issuetype"]
maxResults: 10
```

Display using **Cascading Selection** (4-option limit):

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

### If issue selected or entered

**Ask user (AskUserQuestion):**
"Change issue status to In Progress?"

| Option | Description |
|--------|-------------|
| Yes | Change to In Progress |
| No | Keep current status |

If Yes, transition the issue:
```
Call mcp__atlassian__getTransitionsForJiraIssue
Call mcp__atlassian__transitionJiraIssue with transition to "In Progress"
```

## Step 2: Ask for Description

Ask the user:
"Enter a brief description for the branch (use kebab-case, e.g., emotion-calendar):"

### Description Rules
- Use **kebab-case** (lowercase with hyphens)
- Keep it **short and descriptive** (2-5 words)
- No special characters except hyphens
- Examples: `emotion-calendar`, `auth-redirect-fix`, `readme-update`

**Note:** If Jira issue was selected, you can use the issue summary as the description:
- "Add user authentication" → `add-user-authentication`
- Truncate to 50 characters if needed

## Step 3: Ask for Project Type

Ask the user:
"Is this a collaborative/team project?"

**Options:**
- **Yes**: Include GitHub account name in branch (e.g., `minukHwang/feature/emotion-calendar`)
- **No**: Use simple format (e.g., `feature/emotion-calendar`)

### Get GitHub Account Name (if collaborative)

If collaborative project, get the GitHub username:

```bash
gh api user -q .login
```

If `gh` is not available or fails, ask the user:
"Enter your GitHub username:"

## Step 4: Generate Branch Name

Construct the branch name:

**Without Jira:**
```
{type}/{description}
```

**With Jira issue:**
```
{type}/{issueKey}-{description}
```

**Collaborative without Jira:**
```
{accountName}/{type}/{description}
```

**Collaborative with Jira:**
```
{accountName}/{type}/{issueKey}-{description}
```

### Examples

| Project Type | Jira | Type | Issue | Description | Branch Name |
|--------------|------|------|-------|-------------|-------------|
| Personal | No | feature | - | emotion-calendar | `feature/emotion-calendar` |
| Personal | Yes | feature | CP-1 | add-auth | `feature/CP-1-add-auth` |
| Personal | Yes | bugfix | CP-2 | fix-login | `bugfix/CP-2-fix-login` |
| Collaborative | No | feature | - | user-profile | `minukHwang/feature/user-profile` |
| Collaborative | Yes | hotfix | CP-3 | crash-fix | `minukHwang/hotfix/CP-3-crash-fix` |

## Step 5: Create and Checkout Branch

Create the branch and switch to it:

```bash
git checkout -b <branch-name>
```

## Step 5.5: Update Notion TODO (if Jira linked and notion.enabled)

If a Jira issue was linked and `notion.enabled: true`:

1. Search for TODO item by Jira link:
   Call `mcp__notion__notion-search`:
   ```
   query: "{issueKey}"
   ```

2. If found, update status:
   Call `mcp__notion__notion-update-page`:
   ```
   page_id: {found_page_id}
   properties: {
     "Status": {
       "status": {"name": "In Progress"}
     }
   }
   ```

## Output Format

### On Success (without Jira):
```
✓ Branch created and checked out: <branch-name>

You can now start working on this branch.
When ready, use `/git:commit` to commit your changes.
```

### On Success (with Jira):
```
✓ Branch created and checked out: <branch-name>

Jira Issue: {issueKey}
├─ Summary: {summary}
└─ Status: {old_status} → In Progress

{if notion synced}
✓ Notion TODO status updated: Todo → In Progress
{/if}

You can now start working on this branch.
When ready, use `/git:commit` to commit your changes.
```

### On Failure:
```
✗ Failed to create branch: <error message>
```

### Common Errors:
- **Branch already exists**: "Branch '<name>' already exists. Use a different name or checkout the existing branch with `git checkout <name>`"
- **Invalid branch name**: "Invalid branch name. Please use only lowercase letters, numbers, and hyphens."

## Constraints

1. **Use full type names**: `feature` not `feat`, `bugfix` not `fix`
2. **kebab-case only**: No spaces, underscores, or uppercase
3. **English only**: All branch names in English
4. **Auto-checkout**: Always switch to the new branch after creation
5. **Jira integration optional**: Works without workflow.json
6. **Issue key format**: Uppercase letters + hyphen + number (e.g., CP-123)
