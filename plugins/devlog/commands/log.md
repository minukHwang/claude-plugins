---
description: Record implementation decision to DEVLOG.md with deep analysis
---

# Record Decision Log

Analyze current work and record implementation decisions to DEVLOG.md.

## Step 0: Check Devlog Exists

Check if DEVLOG.md exists:
1. Project root: `DEVLOG.md`
2. Private location: `.claude/devlog/DEVLOG.md`

If not found:
```
⚠️ Devlog not initialized.

Run /devlog:init first to set up devlog files.
```
→ Stop

## Step 1: Deep Analysis

### 1.1 Git Status
```bash
git branch --show-current
git log -1 --format="%h %s" 2>/dev/null || echo "No commits yet"
```

### 1.2 Diff Analysis & Related Files
```bash
git diff --name-only 2>/dev/null
git diff --staged --name-only 2>/dev/null
```

Store the list of changed files as **Related Files** for later use in entry format.

### 1.3 File Content Analysis

**Read changed files with Read tool** to understand:
- What was modified
- Why it might have been modified
- Technical decisions involved

### 1.4 Context Analysis

Extract from conversation:
- Problem being solved
- Options considered
- Decision made
- Rationale

## Step 2: User Confirmation

**Show analysis summary and ask (AskUserQuestion):**
"Review the extracted decision:"

Present:
- Problem: {extracted problem}
- Options: {extracted options}
- Decision: {extracted decision}

| Option | Description |
|--------|-------------|
| Confirm | Proceed with this analysis |
| Edit | Modify the extracted content |

If Edit: Ask user for corrections

## Step 3: Record

### 3.1 Check Mode

Read DEVLOG.md header to determine mode:
- `Mode: public-safe` → Filter sensitive info
- `Mode: private` → No filtering needed

### 3.2 Filter Sensitive Info (if public-safe)

Before writing, replace:
- API keys/tokens: `[REDACTED]`
- Passwords: `[REDACTED]`
- Internal URLs: `[INTERNAL_URL]`
- IP addresses: `[INTERNAL_IP]`
- Secret values: `[REDACTED]`

**Warn user if sensitive content detected:**
```
⚠️ Sensitive info detected and redacted:
- API key in line 5
- Internal URL in line 12
```

### 3.3 Get Current Date/Time
```bash
date '+%Y-%m-%d %H:%M'
```

### 3.4 Append to DEVLOG.md

Use Edit tool to append after the last `---`:

**Format:**
```markdown
### {date} {time} ### {branch}

**Commit**: (empty - will be updated after /git:commit)

**Related Files**
- `{file1}`
- `{file2}`
- `{file3}`

**Context**
{Working context - what you were building/fixing}

**Problem**
{Specific problem that required a decision}

**Options Considered**
| Option | Pros | Cons |
|--------|------|------|
| {option1} | {pros} | {cons} |
| {option2} | {pros} | {cons} |
| **{chosen} (chosen)** | - | - |

**Decision Rationale**
{Why this option was chosen - bullet points}

**Key Implementation Details**
{Important implementation specifics - bullet points}

**Outcome/Lessons**
{Results and learnings - bullet points}

---
```

**Note:** The **Commit** field starts empty. After `/git:commit`, the commit hash will be automatically updated to this entry based on **Related Files** matching.

## Step 4: Confluence Sync (Optional)

Check if Confluence is enabled:
```bash
cat .claude/workflow.json 2>/dev/null | grep -q '"confluence".*"enabled": true'
```

If `confluence.enabled: true`:

**Ask user (AskUserQuestion):**
"Also record to Confluence?"

| Option | Description |
|--------|-------------|
| Yes | Sync to Confluence |
| No | Local only |

### If Yes:

Run `/devlog:jira` to sync this entry to Confluence.

## Step 5: Commit (Optional)

**Ask user (AskUserQuestion):**
"Commit this log?"

| Option | Description |
|--------|-------------|
| Yes | Run /git:commit to commit changes |
| No | Keep as unstaged change |

If Yes → Run `/git:commit`

## Output

### On Success:
```
✓ Decision recorded to DEVLOG.md

Branch: {branch}
Problem: {problem summary}

View: {file path}

Commit this log? [Y/n]
```

### On Failure:
```
✗ Failed to record decision: {error message}
```

## Constraints

1. **Deep analysis**: Always read changed files, don't just rely on diff
2. **User confirmation**: Never record without user approval
3. **Sensitive filtering**: Always check mode and filter if public-safe
4. **Append only**: Never overwrite existing entries
5. **Format consistency**: Follow the exact output format
