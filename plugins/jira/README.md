# Jira Plugin

Jira issue management and Git integration automation.

## Version

1.0.0

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

## Features

### Issue Management
- Create issues with type selection (Task, Bug, Story, Epic)
- View backlog and assigned issues
- Track issue status throughout workflow

### Git Integration
- Automatic branch creation with issue key
- Issue key included in commit messages
- PR creation with Jira link

### Notion Sync
- Create TODO items in Notion when creating Jira issues
- Update Notion status and Start Date when starting issues
- Add commit and PR links on completion

## Workflow Example

```
/jira:create "Add user authentication"
    |
    v
Creates Jira issue (CP-1) + Notion TODO item
    |
    v
/jira:start CP-1
    |
    v
Creates branch (feature/CP-1-add-user-authentication)
Updates Jira status (To Do -> In Progress)
Updates Notion (Status: Todo -> In Progress, Start Date: today)
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
Updates Notion (status + commit link + PR link)
```

## Configuration

### Project Config: `.claude/workflow.json`

```json
{
  "jira": {
    "enabled": true,
    "cloudId": "your-site.atlassian.net",
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

Notion TODO DB ID is stored at user-level:

```json
{
  "todo": {
    "id": "xxx",
    "name": "[CLAUDE] TODO"
  }
}
```

**Note:** TODO DB is configured by `/workflow:init`

## Requirements

- Atlassian MCP (mcp__atlassian__*)
- Notion MCP (mcp__notion__*) - optional, for TODO sync
- workflow plugin configured
