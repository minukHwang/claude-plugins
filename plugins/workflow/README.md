# Workflow Plugin

Project workflow configuration for Git strategy, Jira integration, and Notion sync.

## Version

1.1.0

## Commands

| Command | Description |
|---------|-------------|
| `/workflow:init` | Initialize workflow configuration for a project |
| `/workflow:config` | Modify existing workflow settings |

## Features

### Git Strategy Configuration
- **github-flow**: main + feature branches (default)
- **git-flow**: main + develop + feature/release/hotfix
- **trunk-based**: main only, short-lived branches

### Jira Integration
- Connect to Jira project
- Auto-include issue keys in branch names
- Auto-include issue keys in commit messages

### Notion Integration
- Connect [CLAUDE] TODO DB for task tracking
- **TODO DB with Epic field**: Links Jira Epic info to Notion (v1.1.0)
- TODO DB stored in user-level config (`~/.claude/notion.json`)
- TIL/BLOG DBs configured by `/notion:til` and `/notion:blog` commands

### Merge Method
- **squash**: Squash commits into one (recommended)
- **merge**: Regular merge commit
- **rebase**: Rebase then merge

## Configuration Files

### Project Config: `.claude/workflow.json`

**⚠️ Add to `.gitignore`** - contains personal Atlassian/project settings:
```
.claude/workflow.json
```

```json
{
  "version": "1.0",
  "git": {
    "strategy": "github-flow",
    "branches": { "main": "main", "develop": "develop" },
    "merge": { "method": "squash", "deleteOnMerge": true },
    "pr": { "defaultTarget": "main" }
  },
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
  },
  "confluence": {
    "enabled": false,
    "spaceKey": null
  }
}
```

### User Config: `~/.claude/notion.json`

Notion DB IDs are stored at user-level (shared across projects):

```json
{
  "todo": {
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "pageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "[CLAUDE] TODO"
  },
  "til": {
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "pageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "[CLAUDE] TIL"
  },
  "blog": {
    "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "pageId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "name": "[CLAUDE] BLOG"
  }
}
```

**ID Types (v1.1.0):**
- `id`: Data Source ID (collection://) - for creating pages
- `pageId`: Database Page ID - for schema updates

**Note:**
- TODO DB is configured by `/workflow:init`
- TIL DB is configured by `/notion:til` (first run)
- BLOG DB is configured by `/notion:blog` (first run)

## Usage

### Initialize Workflow

```
/workflow:init
```

This will guide you through:
1. Selecting Git strategy
2. Connecting Jira project (optional)
3. Configuring issue number inclusion
4. Selecting merge method
5. Connecting Notion databases (optional)

### Modify Settings

```
/workflow:config
```

Change individual settings without re-initializing.

## Integration with Other Plugins

The workflow configuration is read by:
- **git plugin**: Uses Jira settings for branch/commit naming
- **jira plugin**: Uses project settings for issue management
- **devlog plugin**: Uses Confluence settings for sync
- **notion plugin**: Uses database IDs for TIL/Blog

## Requirements

- Atlassian MCP (for Jira integration)
- Notion MCP (for Notion integration)
