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

## Step 3: User-Scope Skill Installation (Optional)

**Ask user (AskUserQuestion):**
"Install devlog-reminder skill globally?"

| Option | Description |
|--------|-------------|
| Yes | Copy skill to ~/.claude/skills/ (works across all projects) |
| No | Skip (use plugin's bundled skill only) |

### If Yes:

#### 3.1 Create skill directory
```bash
mkdir -p ~/.claude/skills/devlog-reminder
```

#### 3.2 Copy skill file

Use Write tool to create `~/.claude/skills/devlog-reminder/SKILL.md`:

```markdown
---
name: devlog-reminder
description: Use after completing design decisions or implementation work - suggests /devlog:plan for design or /devlog:log for implementation to record decisions
---

# Devlog Reminder

Suggest devlog recording after completing design decisions or implementation work.

## Trigger Conditions

When the following work is **completed**:

| Work Type | Examples | Command to Suggest |
|-----------|----------|-------------------|
| Design decisions | Architecture choices, library comparisons, pattern decisions | `/devlog:plan` |
| Implementation work | Feature implementation, bug fixes, refactoring | `/devlog:log` |

## Suggestion Format

Suggest naturally after work completion:

\`\`\`
[Work completion message]

💡 Record this decision/implementation?
- `/devlog:plan` - Record design decision
- `/devlog:log` - Record implementation
\`\`\`

## Do NOT Suggest When

- Simple Q&A only
- Just reading files without changes
- Already ran devlog command in this session
- Trivial changes (typo fixes, adding comments, etc.)

## Decision Criteria

**Is it worth recording?**
- Would someone ask "why did we do it this way?" later?
- Is there context other developers should know?
- Were there trade-offs involved?

If any of the above applies, suggest devlog.
```

## Step 4: PreCompact Hook Setup (Optional)

**Ask user (AskUserQuestion):**
"Set up PreCompact hook for devlog reminder?"

| Option | Description |
|--------|-------------|
| Yes | Add hook to ~/.claude/settings.json |
| No | Skip hook setup |

### If Yes:

#### 4.1 Read existing settings
```bash
cat ~/.claude/settings.json 2>/dev/null || echo "{}"
```

#### 4.2 Merge PreCompact hook

Read `~/.claude/settings.json` with Read tool, then merge:

```json
{
  "hooks": {
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

#### 4.3 Write updated settings

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

{if skill installed}
Skill installed:
- ~/.claude/skills/devlog-reminder/SKILL.md
{/if}

{if hook set up}
Hook configured:
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
