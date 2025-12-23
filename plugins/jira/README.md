# Jira Plugin

Jira issue management and Git integration automation.

## Version

1.3.1

## Prerequisites

- Atlassian MCP server configured
- `/workflow:init` completed with Jira enabled

## Commands

| Command | Description |
|---------|-------------|
| `/jira:backlog` | View project backlog issues |
| `/jira:create` | Create a new Jira issue |
| `/jira:start` | Start working on an issue (creates branch) |
| `/jira:done` | Complete issue (creates PR, updates status) |
| `/jira:view` | View issue details |
| `/jira:sync` | Bidirectional sync: Jira ↔ Notion TODO (v1.3.0) |

## Features

### Issue Management
- Create issues with type selection (Task, Bug, Story, Epic)
- **Epic selection**: Link issues to parent Epics (v1.1.0)
- **Period**: Start/Due date as range, synced to Notion calendar (v1.3.1)
- **Auto-assign**: Issues assigned to current user (v1.1.0)
- View backlog and assigned issues
- Track issue status throughout workflow

### Git Integration
- Automatic branch creation with issue key
- Issue key included in commit messages
- PR creation with Jira link
- **1 PR = 1 Issue**: Warning on multi-issue PRs (v1.3.0)

### Notion Sync
- **Type field**: Epic/Story/Task/Bug synced (v1.3.0)
- Create TODO items in Notion when creating Jira issues
- **Epic field**: Rich text link to Epic (v1.3.0)
- **Period field**: Start → End date range for calendar view (v1.3.1)
- Update Notion status when starting issues
- Add PR links on completion (TIL format: `[#42](url)`)
- **Bidirectional sync**: Jira ↔ Notion via `/jira:sync` (v1.3.0)
  - Done status propagation
  - Project scope selection

## Notion TODO Fields

| Field | Type | Description |
|-------|------|-------------|
| Title | title | Issue summary |
| Type | select | Epic/Story/Task/Bug/Todo |
| Epic | rich_text | `[KEY](url) - Summary` |
| Status | status | Todo/In Progress/In Review/Done |
| Priority | select | High/Medium/Low |
| Project | rich_text | `[repo](url)` |
| Jira Link | rich_text | `[KEY](url)` |
| PR | rich_text | `[#number](url)` |
| Period | date | Start → End (calendar range) |

**Note:** Type = "Todo" items are personal and excluded from Jira sync.

## Workflow Example

```
/jira:create "Add user authentication"
    |
    v
Creates Jira issue (CP-1) + Notion TODO item
(Type: Task, Epic: linked, Period: optional)
    |
    v
/jira:start CP-1
    |
    v
Creates branch (feature/CP-1-add-user-authentication)
Updates Jira status (To Do -> In Progress)
Sets Period:start (if not already set)
Updates Notion (Status, Period)
    |
    v
Work on feature, commit with [CP-1]
    |
    v
/jira:done
    |
    v
Creates PR with Jira link
Updates Jira status (In Progress -> In Review/Done)
Updates Notion (Status, PR link)
```

## Configuration

### Project Config: `.claude/workflow.json`

**⚠️ Add to `.gitignore`** - contains personal settings

```json
{
  "jira": {
    "enabled": true,
    "cloudId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "siteUrl": "yoursite.atlassian.net",
    "projectKey": "PROJ",
    "includeInBranch": true,
    "includeInCommit": true
  },
  "notion": {
    "enabled": true
  }
}
```

### User Config: `~/.claude/notion.json`

Notion TODO DB IDs are stored at user-level:

```json
{
  "todo": {
    "id": "xxx",
    "pageId": "yyy",
    "name": "[CLAUDE] TODO"
  }
}
```

**ID Types:**
- `id`: Data Source ID (collection://) - for creating pages
- `pageId`: Database Page ID - for schema updates

**Note:** TODO DB is configured by `/workflow:init`

## Requirements

- Atlassian MCP (mcp__atlassian__*)
- Notion MCP (mcp__notion__*) - optional, for TODO sync
- workflow plugin configured
