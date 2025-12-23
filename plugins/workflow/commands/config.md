---
description: Modify existing workflow settings
---

# Configure Workflow

Modify individual workflow settings without re-initializing.

## Step 0: Check Existing Configuration

```bash
cat .claude/workflow.json 2>/dev/null
```

If not exists, inform user:
```
✗ No workflow configuration found.
Run /workflow:init to create a new configuration.
```

## Step 1: Select Setting to Modify

**Ask user (AskUserQuestion):**
"Which setting would you like to modify?"

| Option | Description |
|--------|-------------|
| Git Strategy | Change branching strategy |
| Jira Settings | Modify Jira integration |
| Notion Settings | Modify Notion integration |
| View Current | Display current configuration |

## Step 2: Modify Selected Setting

### If Git Strategy

**Ask user (AskUserQuestion):**
"Select the Git branching strategy:"

| Option | Description |
|--------|-------------|
| github-flow | main + feature branches (Recommended) |
| git-flow | main + develop + feature/release/hotfix |
| trunk-based | main only, short-lived branches |

**Ask user (AskUserQuestion):**
"Select the preferred merge method for PRs:"

| Option | Description |
|--------|-------------|
| squash | Squash commits into one (Recommended) |
| merge | Create regular merge commit |
| rebase | Rebase then merge |

Update `git` section in workflow.json.

---

### If Jira Settings

**Ask user (AskUserQuestion):**
"What would you like to change?"

| Option | Description |
|--------|-------------|
| Enable/Disable | Toggle Jira integration |
| Change Project | Select different project |
| Issue Inclusion | Modify branch/commit settings |

#### If Enable/Disable

Toggle `jira.enabled` in workflow.json.

#### If Change Project

Call `mcp__atlassian__getVisibleJiraProjects` to get project list.

Display projects using **Cascading Selection** (4-option limit):

**Round 1:**
| Option | Description |
|--------|-------------|
| {project1.key} | {project1.name} |
| {project2.key} | {project2.name} |
| {project3.key} | {project3.name} |
| More... | Show more projects |

**Round 2 (if More...):**
| Option | Description |
|--------|-------------|
| {project4.key} | {project4.name} |
| {project5.key} | {project5.name} |
| {project6.key} | {project6.name} |
| Other | Enter project key manually |

Update `jira.projectKey` in workflow.json.

#### If Issue Inclusion

**Ask user (AskUserQuestion):**
"How should Jira issue numbers be included?"

| Option | Description |
|--------|-------------|
| Branch + Commit | feature/CP-1-desc + [CP-1] in commit |
| Branch only | feature/CP-1-desc |
| Commit only | [CP-1] in commit message |
| None | Don't include issue numbers |

Update `jira.includeInBranch` and `jira.includeInCommit` in workflow.json.

---

### If Notion Settings

**Ask user (AskUserQuestion):**
"What would you like to change?"

| Option | Description |
|--------|-------------|
| Enable/Disable | Toggle Notion integration |
| TODO DB | Change TODO database |
| TIL DB | Change TIL database |
| BLOG DB | Change BLOG database |

#### If Enable/Disable

Toggle `notion.enabled` in workflow.json.

#### If TODO/TIL/BLOG DB

Search for databases:
- Call `mcp__notion__notion-search` with query "[CLAUDE]"

**Ask user (AskUserQuestion):**
"Select a database or enter ID:"

| Option | Description |
|--------|-------------|
| {db1.title} | {db1.id} |
| {db2.title} | {db2.id} |
| {db3.title} | {db3.id} |
| Enter ID | Provide database ID manually |

Update corresponding database ID in workflow.json.

---

### If View Current

Display current configuration:

```
Current Workflow Configuration
==============================

Git Strategy: {strategy}
├─ Main branch: {main}
├─ Develop branch: {develop}
└─ Merge method: {merge_method}

Jira: {enabled/disabled}
{if enabled}
├─ Cloud ID: {cloudId}
├─ Project: {projectKey}
├─ Include in branch: {yes/no}
└─ Include in commit: {yes/no}
{/if}

Notion: {enabled/disabled}
{if enabled}
├─ TODO DB: {todo.name} ({todo.id})
├─ TIL DB: {til.name} ({til.id})
└─ BLOG DB: {blog.name} ({blog.id})
{/if}

Confluence: {enabled/disabled}
{if enabled}
└─ Space Key: {spaceKey}
{/if}
```

## Step 3: Save Configuration

Update `.claude/workflow.json` with modified settings.

## Output

### On Success:
```
✓ Configuration updated!

Changed: {setting_name}
├─ Old: {old_value}
└─ New: {new_value}

File: .claude/workflow.json
```

### On Failure:
```
✗ Failed to update configuration: {error message}
```

## Constraints

1. **Validate before save**: Ensure all values are valid
2. **Preserve other settings**: Only modify the selected setting
3. **Backup consideration**: Warn user before major changes
4. **JSON formatting**: Write valid, formatted JSON
