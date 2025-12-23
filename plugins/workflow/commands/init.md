---
description: Initialize workflow configuration for a project (Git strategy + Jira + Notion TODO)
---

# Initialize Workflow

Set up project workflow configuration with Git strategy, Jira integration, and Notion TODO sync.

## Step 0: Check Existing Configuration

```bash
cat .claude/workflow.json 2>/dev/null
```

If exists, ask user:

**Ask user (AskUserQuestion):**
"Workflow configuration already exists. What would you like to do?"

| Option | Description |
|--------|-------------|
| Reconfigure | Start fresh with new settings |
| Keep existing | Exit without changes |

If "Keep existing", exit with message: "Existing configuration preserved."

## Step 1: Select Git Strategy

**Ask user (AskUserQuestion):**
"Select the Git branching strategy for this project:"

| Option | Description |
|--------|-------------|
| github-flow | main + feature branches (Recommended) |
| git-flow | main + develop + feature/release/hotfix |
| trunk-based | main only, short-lived branches |

### Branch Configuration (if git-flow)

**Ask user (AskUserQuestion):**
"What is your main branch name?"

| Option | Description |
|--------|-------------|
| main | Use 'main' as production branch |
| master | Use 'master' as production branch |

## Step 2: Jira Integration

**Ask user (AskUserQuestion):**
"Would you like to connect a Jira project?"

| Option | Description |
|--------|-------------|
| Yes | Connect Jira project |
| No | Use Git only |

### Step 2.1: Select Jira Project (if Yes)

First, get Atlassian cloud info:
```
mcp__atlassian__getAccessibleAtlassianResources
```

Extract from response:
- `{cloud_id}`: The `id` field (UUID)
- `{site_url}`: The `url` field without `https://` (e.g., `yoursite.atlassian.net`)

Then call `mcp__atlassian__getVisibleJiraProjects` with the cloudId to get project list.

Display projects using **Cascading Selection** (4-option limit):

**Round 1:**
Show first 3 projects + "More..."

| Option | Description |
|--------|-------------|
| {project1.key} | {project1.name} |
| {project2.key} | {project2.name} |
| {project3.key} | {project3.name} |
| More... | Show more projects |

**Round 2 (if More...):**
Show next 3 projects + "Other"

| Option | Description |
|--------|-------------|
| {project4.key} | {project4.name} |
| {project5.key} | {project5.name} |
| {project6.key} | {project6.name} |
| Other | Enter project key manually |

**Note:** Project list is dynamically generated from MCP query, not hardcoded.

## Step 3: Issue Number Inclusion (if Jira enabled)

**Ask user (AskUserQuestion):**
"How should Jira issue numbers be included?"

| Option | Description |
|--------|-------------|
| Branch + Commit | feature/CP-1-desc + [CP-1] in commit |
| Branch only | feature/CP-1-desc |
| Commit only | [CP-1] in commit message |
| None | Don't include issue numbers |

## Step 4: Select Merge Method

**Ask user (AskUserQuestion):**
"Select the preferred merge method for PRs:"

| Option | Description |
|--------|-------------|
| squash | Squash commits into one (Recommended) |
| merge | Create regular merge commit |
| rebase | Rebase then merge |

## Step 5: Notion TODO Integration

**Ask user (AskUserQuestion):**
"Would you like to connect Notion TODO database for task tracking?"

| Option | Description |
|--------|-------------|
| Yes | Configure Notion TODO DB |
| No | Skip Notion integration |

### Step 5.1: Check/Search/Create TODO DB (if Yes)

**Flow:**
```
1. Check ~/.claude/notion.json for existing TODO config
2. If not configured → Search Notion for "[CLAUDE] TODO"
3. If not found → Create new DB in Notion
4. Save to ~/.claude/notion.json
```

#### 5.1.1: Check User Config

```bash
cat ~/.claude/notion.json 2>/dev/null
```

If `todo.id` exists in config → Use existing, skip to Step 6

#### 5.1.2: Search Notion for Existing DB

Call `mcp__notion__notion-search`:
```
query: "[CLAUDE] TODO"
filter: { "property": "object", "value": "database" }
```

If found:
- Extract database ID
- Save to `~/.claude/notion.json`
- Display: "Found existing [CLAUDE] TODO database"
- Skip to Step 6

#### 5.1.3: Create New TODO DB (if not found)

**Ask user (AskUserQuestion):**
"[CLAUDE] TODO database not found. What would you like to do?"

| Option | Description |
|--------|-------------|
| Create new | Create new [CLAUDE] TODO database |
| Skip | Don't use TODO tracking |
| Enter ID | Provide existing database ID manually |

**If Create new:**

Call `mcp__notion__notion-create-database`:
```
parent: { "type": "workspace", "workspace": true }
title: "[CLAUDE] TODO"
properties: {
  "Title": { "title": {} },
  "Type": {
    "select": {
      "options": [
        { "name": "Epic", "color": "purple" },
        { "name": "Story", "color": "green" },
        { "name": "Task", "color": "blue" },
        { "name": "Bug", "color": "red" },
        { "name": "Todo", "color": "gray" }
      ]
    }
  },
  "Epic": { "rich_text": {} },
  "Status": {
    "status": {
      "options": [
        { "name": "Todo", "color": "default" },
        { "name": "In Progress", "color": "blue" },
        { "name": "In Review", "color": "yellow" },
        { "name": "Done", "color": "green" }
      ]
    }
  },
  "Project": { "rich_text": {} },
  "Jira Link": { "rich_text": {} },
  "PR": { "rich_text": {} },
  "Priority": {
    "select": {
      "options": [
        { "name": "High", "color": "red" },
        { "name": "Medium", "color": "yellow" },
        { "name": "Low", "color": "gray" }
      ]
    }
  },
  "Start Date": { "date": {} },
  "Due Date": { "date": {} }
}
```

#### 5.1.4: Save to User Config

Create `~/.claude` directory if not exists:
```bash
mkdir -p ~/.claude
```

Read existing config or create new:
```bash
cat ~/.claude/notion.json 2>/dev/null || echo '{}'
```

Update with TODO DB info and write:

```json
{
  "todo": {
    "id": "{data_source_id}",
    "pageId": "{database_page_id}",
    "name": "[CLAUDE] TODO"
  }
}
```

**ID Types:**
- `id`: Data Source ID (collection://) - for creating pages
- `pageId`: Database Page ID - for schema updates

**Note:** TIL and BLOG DBs are configured separately by `/notion:til` and `/notion:blog` commands.

## Step 6: Create Project Configuration File

Create `.claude` directory if not exists:
```bash
mkdir -p .claude
```

Write `.claude/workflow.json` with collected settings:

```json
{
  "version": "1.0",
  "git": {
    "strategy": "{selected_strategy}",
    "branches": {
      "main": "{main_branch}",
      "develop": "develop"
    },
    "merge": {
      "method": "{merge_method}",
      "deleteOnMerge": true
    },
    "pr": {
      "defaultTarget": "{main_branch}"
    }
  },
  "jira": {
    "enabled": {jira_enabled},
    "cloudId": "{cloud_id}",
    "siteUrl": "{site_url}",
    "projectKey": "{project_key}",
    "includeInBranch": {include_in_branch},
    "includeInCommit": {include_in_commit}
  },
  "notion": {
    "enabled": {notion_enabled}
  },
  "confluence": {
    "enabled": false,
    "spaceKey": null
  }
}
```

**Note:** Notion DB IDs are stored in `~/.claude/notion.json` (user-level), not in project workflow.json.

## Step 7: Add Jira Workflow Rules to CLAUDE.md (if Jira enabled)

If Jira integration is enabled, append workflow rules to project's CLAUDE.md:

```bash
cat CLAUDE.md 2>/dev/null
```

If CLAUDE.md exists and doesn't contain "Jira Workflow Rules", append:

```markdown
## Jira Workflow Rules

- **1 PR = 1 Issue**: Each PR should contain only one Jira issue
- **Branch naming**: `{type}/{ISSUE-KEY}-description` (e.g., `feature/CP-1-add-auth`)
- **New task during work**: Stash changes → /jira:start NEW-ISSUE → Complete → Return
- **Commit messages**: Include issue key `[CP-1] commit message`
```

If CLAUDE.md doesn't exist, create it with the rules above.

## Output

### On Success:
```
✓ Workflow initialized!

Project Config: .claude/workflow.json
User Config: ~/.claude/notion.json

Git Strategy: {strategy}
├─ Main branch: {main}
└─ Merge method: {merge_method}

Jira: {enabled/disabled}
{if enabled}
├─ Project: {project_key}
├─ Include in branch: {yes/no}
└─ Include in commit: {yes/no}
{/if}

Notion TODO: {connected/not configured}
{if connected}
└─ DB: [CLAUDE] TODO ({db_id})
{/if}

{if jira enabled}
CLAUDE.md: Jira Workflow Rules added
└─ 1 PR = 1 Issue workflow enforced
{/if}

Next steps:
- /jira:create - Create a new Jira issue
- /jira:start - Start working on an issue
- /git:branch - Create a branch (with Jira integration)
- /notion:til - Configure TIL database (first use)
- /notion:blog - Configure BLOG database (first use)
```

### On Failure:
```
✗ Failed to initialize workflow: {error message}
```

## Constraints

1. **Don't overwrite silently**: Always ask before replacing existing config
2. **Validate Jira connection**: Ensure MCP connection works before saving
3. **Validate Notion connection**: Ensure MCP connection works before saving
4. **User-level Notion config**: Store DB IDs in `~/.claude/notion.json`
5. **Project-level workflow config**: Store workflow settings in `.claude/workflow.json`
6. **TODO only**: TIL/BLOG DBs are handled by their respective plugins
