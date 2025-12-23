---
description: Sync devlog entries to Confluence
---

# Sync Devlog to Confluence

Sync DEVLOG.md and PLANS.md entries to a Confluence page.

## Step 0: Load Workflow Configuration

```bash
cat .claude/workflow.json 2>/dev/null
```

If not exists or `confluence.enabled` is false:
```
✗ Confluence integration not configured.
Run /workflow:init to set up Confluence integration.
```

Extract:
- `confluence.spaceKey` - Confluence space key
- `jira.cloudId` - Atlassian cloud ID

## Step 1: Check/Create Confluence Page

### 1.1 Get Project Name

```bash
basename $(pwd)
```

### 1.2 Search for Existing Page

Search for "[{Project Name}] Development Log" page:

Call `mcp__atlassian__searchConfluenceUsingCql`:
```
cloudId: {cloudId}
cql: "title ~ '[{projectName}] Development Log' AND space = {spaceKey}"
```

### 1.3 Create Page if Not Exists

If page not found:

Call `mcp__atlassian__createConfluencePage`:
```
cloudId: {cloudId}
spaceId: {spaceId}
title: "[{projectName}] Development Log"
body: |
  # Development Log

  This page contains development decisions and logs for the {projectName} project.

  ## Recent Entries

  (Entries will be added here)

  ---

  *Auto-synced from local DEVLOG.md and PLANS.md*
contentFormat: "markdown"
```

Store `pageId` for updates.

## Step 2: Read Local Devlog Files

### 2.1 Read DEVLOG.md

Check locations:
1. `DEVLOG.md`
2. `.claude/devlog/DEVLOG.md`

Read file content with Read tool.

### 2.2 Read PLANS.md

Check locations:
1. `PLANS.md`
2. `.claude/devlog/PLANS.md`

Read file content with Read tool.

## Step 3: Select Sync Mode

**Ask user (AskUserQuestion):**
"Select sync mode:"

| Option | Description |
|--------|-------------|
| Latest entry | Sync only the most recent entry |
| Full sync | Sync all entries |
| Select entries | Choose specific entries |

### If Latest entry

Extract the last entry from each file (between last `---` markers).

### If Full sync

Use complete file contents.

### If Select entries

List available entries (by date/branch) and let user select.

## Step 4: Format Content for Confluence

Convert markdown to Confluence-compatible format:

**Transformation rules:**
- Convert tables to Confluence table format
- Preserve headings (`###` → `h3.`)
- Convert code blocks (triple backticks → code macro)
- Convert bullet lists

**Structure:**
```markdown
## Implementation Decisions (DEVLOG)

{devlog entries}

---

## Design Decisions (PLANS)

{plans entries}

---

*Last synced: {current_date_time}*
```

## Step 5: Update Confluence Page

Call `mcp__atlassian__updateConfluencePage`:
```
cloudId: {cloudId}
pageId: {pageId}
body: {formatted_content}
contentFormat: "markdown"
versionMessage: "Synced devlog entries"
```

## Output

### On Success:
```
✓ Devlog synced to Confluence

Page: [{projectName}] Development Log
URL: https://{cloudId}/wiki/spaces/{spaceKey}/pages/{pageId}

Synced:
├─ DEVLOG.md: {entry_count} entries
└─ PLANS.md: {entry_count} entries

Last sync: {date_time}
```

### On Page Not Found:
```
✓ Created new Confluence page

Page: [{projectName}] Development Log
URL: https://{cloudId}/wiki/spaces/{spaceKey}/pages/{pageId}

Initial sync completed.
```

### On Failure:
```
✗ Failed to sync to Confluence: {error message}
```

## Constraints

1. **Check configuration first**: Ensure workflow.json has Confluence enabled
2. **Create page if missing**: Auto-create the development log page
3. **Preserve existing content**: Append/update, don't overwrite entire page
4. **Format conversion**: Properly convert markdown to Confluence format
5. **Version messages**: Include descriptive version messages for history
