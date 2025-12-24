---
description: Sync a file to Confluence
---

# Sync File to Confluence

Sync a specified file to a Confluence page.

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

## Step 1: Get File Path

### If argument provided:
Use the provided file path directly.

### If no argument:
**Ask user (AskUserQuestion):**
"Enter file path to sync:"

Validate file exists with Read tool.

## Step 2: Get Project and File Info

```bash
basename $(pwd)
```

Extract filename from path (without extension for page title).

## Step 3: Search for Existing Page

Search for "[{Project}] {filename}" page:

Call `mcp__atlassian__searchConfluenceUsingCql`:
```
cloudId: {cloudId}
cql: "title ~ '[{projectName}] {filename}' AND space = {spaceKey}"
```

## Step 4: Read File Content

Read the file using Read tool.

## Step 5: Create or Update Page

### If page not found:

Call `mcp__atlassian__createConfluencePage`:
```
cloudId: {cloudId}
spaceId: {spaceId}
title: "[{projectName}] {filename}"
body: {file_content}
contentFormat: "markdown"
```

### If page exists:

Call `mcp__atlassian__updateConfluencePage`:
```
cloudId: {cloudId}
pageId: {pageId}
body: {file_content}
contentFormat: "markdown"
versionMessage: "Synced from local file"
```

## Output

### On Success:
```
✓ File synced to Confluence

File: {file_path}
Page: [{projectName}] {filename}
URL: https://{domain}/wiki/spaces/{spaceKey}/pages/{pageId}

Last sync: {date_time}
```

### On Page Created:
```
✓ Created new Confluence page

File: {file_path}
Page: [{projectName}] {filename}
URL: https://{domain}/wiki/spaces/{spaceKey}/pages/{pageId}
```

### On Failure:
```
✗ Failed to sync to Confluence: {error message}
```

## Constraints

1. **Check configuration first**: Ensure workflow.json has Confluence enabled
2. **Validate file exists**: File must exist before syncing
3. **Simple page naming**: `[{Project}] {filename}` format
4. **Full content sync**: Replace entire page content with file content
