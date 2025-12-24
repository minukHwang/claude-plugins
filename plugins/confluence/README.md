# Confluence Plugin

Sync files to Confluence.

## Version

1.0.0

## Purpose

Simple file sync to Confluence pages. Specify a file path, and it gets uploaded to Confluence.

## Commands

| Command | Description |
|---------|-------------|
| `/confluence:sync` | Sync a file to Confluence |

## Quick Start

```bash
# Sync a specific file
/confluence:sync docs/plans/PLAN_feature-a.md

# Or run without arguments to be prompted for file path
/confluence:sync
```

## Requirements

- `.claude/workflow.json` with Confluence enabled
- Confluence MCP configured

## Usage

1. **With file path argument:**
   ```bash
   /confluence:sync <file_path>
   ```

2. **Without arguments:**
   ```bash
   /confluence:sync
   # → "Enter file path to sync:"
   # → User provides path
   ```

## Page Naming

Pages are created with the format: `[{Project}] {filename}`

Example: `[claude-plugins] PLAN_feature-a`

## Features

- Creates new page if not exists
- Updates existing page if found
- Returns Confluence page URL after sync
