---
description: Initialize devlog files and optional global hooks
---

# Initialize Devlog

Set up project decision logging with optional git integration and auto-trigger hooks.

## Step 1: Git Inclusion Choice

**Ask user (AskUserQuestion):**
"Include devlog in git?"

| Option | Description |
|--------|-------------|
| Yes (public-safe) | Store in project root, filter sensitive info |
| No (private) | Store in `.claude/devlog/`, add to .gitignore |

### If public-safe mode:
- Location: Project root (`DEVLOG.md`, `PLANS.md`)
- Sensitive info will be filtered in all logs
- Files will be committed with code

### If private mode:
- Location: `.claude/devlog/DEVLOG.md`, `.claude/devlog/PLANS.md`
- Add `.claude/devlog/` to `.gitignore`
- Personal notes only

## Step 2: Create Project Files

### 2.1 Create Directory (if private mode)
```bash
mkdir -p .claude/devlog
```

### 2.2 Create DEVLOG.md

Use Write tool to create:

**Template:**
```markdown
# Development Log

Project decision records.

<!--
Mode: {public-safe | private}
Created: {date}
-->

---

```

### 2.3 Create PLANS.md

Use Write tool to create:

**Template:**
```markdown
# Design Decisions

Design phase decision records.

<!--
Mode: {public-safe | private}
Created: {date}
-->

---

```

### 2.4 Update .gitignore (if private mode)

Check if `.gitignore` exists, then append:
```
# Devlog (private mode)
.claude/devlog/
```

## Step 3: Global Hooks Setup (Optional)

**Ask user (AskUserQuestion):**
"Set up auto-trigger hooks globally? (Recommended for first-time setup)"

| Option | Description |
|--------|-------------|
| Yes | Add Stop + PreCompact hooks to ~/.claude/settings.json |
| No | Skip hook setup |

### If Yes:

#### 3.1 Read existing settings
```bash
cat ~/.claude/settings.json 2>/dev/null || echo "{}"
```

#### 3.2 Merge hooks config

Read `~/.claude/settings.json` with Read tool, then merge the following hooks:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "After this response, check: 1) If ExitPlanMode was just called, suggest /devlog:plan. 2) If significant decisions were made, suggest /devlog:log. Only if relevant."
      }]
    }],
    "PreCompact": [{
      "matcher": "auto",
      "hooks": [{
        "type": "command",
        "command": "echo '\\n💡 Before compact: Run /devlog:plan (if designing) or /devlog:log (if coding)\\n'"
      }]
    }]
  }
}
```

#### 3.3 Write updated settings

Use Write tool to save merged config to `~/.claude/settings.json`

**Note:** Preserve existing hooks - merge, don't replace.

## Output

### On Success:
```
✓ Devlog initialized!

Mode: {public-safe | private}
Location: {path}

Files created:
- DEVLOG.md
- PLANS.md

{if hooks set up}
Hooks configured:
- Stop: Auto-suggest after Plan mode / decisions
- PreCompact: Reminder before context compaction
{/if}

Next steps:
- /devlog:log - Record implementation decisions
- /devlog:plan - Record design decisions
- /devlog:review - Analyze decision history
```

### On Failure:
```
✗ Failed to initialize devlog: {error message}
```

## Constraints

1. **Check existing files**: Don't overwrite if DEVLOG.md or PLANS.md already exist
2. **Preserve settings**: Merge hooks, don't replace entire settings.json
3. **Git safety**: Only modify .gitignore in private mode
