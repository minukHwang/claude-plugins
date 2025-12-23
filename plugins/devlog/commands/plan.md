---
description: Record design decision to PLANS.md after Plan mode
---

# Record Design Decision

Analyze Plan mode discussion and record design decisions to PLANS.md.

## Step 0: Check Devlog Exists

Check if PLANS.md exists:
1. Project root: `PLANS.md`
2. Private location: `.claude/devlog/PLANS.md`

If not found:
```
⚠️ Devlog not initialized.

Run /devlog:init first to set up devlog files.
```
→ Stop

## Step 1: Deep Analysis (Plan Mode Context)

### 1.1 Branch Info
```bash
git branch --show-current
```

### 1.2 Plan File Analysis

Check for plan files:
```bash
ls ~/.claude/plans/*.md 2>/dev/null
```

If found, read with Read tool to extract:
- Design goals
- Approaches considered
- Selected approach
- Trade-offs

### 1.3 Conversation Context Analysis

Extract from conversation:
- Design goal / requirements
- Architectural options considered
- Chosen approach
- Trade-offs accepted
- Implementation scope

### 1.4 Related Files

Identify files to be created/modified based on plan

## Step 2: User Confirmation

**Show analysis summary and ask (AskUserQuestion):**
"Review the extracted design decision:"

Present:
- Goal: {extracted goal}
- Approaches: {extracted approaches}
- Decision: {chosen approach}
- Trade-offs: {extracted trade-offs}

| Option | Description |
|--------|-------------|
| Confirm | Proceed with this analysis |
| Edit | Modify the extracted content |

If Edit: Ask user for corrections

## Step 3: Record

### 3.1 Check Mode

Read PLANS.md header to determine mode:
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
- Internal URL in architecture section
```

### 3.3 Get Current Date/Time
```bash
date '+%Y-%m-%d %H:%M'
```

### 3.4 Append to PLANS.md

Use Edit tool to append after the last `---`:

**Format:**
```markdown
### {date} {time} ### {branch}

**Design Goal**
{Clear statement of what we're trying to achieve and constraints}

**Requirements**
{Bullet list of requirements}

**Approaches Considered**
| Approach | Pros | Cons |
|----------|------|------|
| {approach1} | {pros} | {cons} |
| {approach2} | {pros} | {cons} |
| **{chosen} (chosen)** | - | - |

**Decision Rationale**
{Why this approach was chosen - bullet points}

**Architecture Decisions**
{Key architectural choices - bullet points}

**Trade-offs Accepted**
{What we're giving up and why it's acceptable}

**Implementation Scope**
| Component | Description |
|-----------|-------------|
| {component1} | {description} |
| {component2} | {description} |

**Files to Create/Modify**
{List of files with create/modify indicator}

**Open Questions**
{Remaining questions to resolve during implementation}

---
```

**Note:** No commit prompt - design decisions will be committed with implementation.

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

## Output

### On Success:
```
✓ Design decision recorded to PLANS.md

Branch: {branch}
Goal: {goal summary}

View: {file path}

Note: Will be committed together with implementation.
```

### On Failure:
```
✗ Failed to record design decision: {error message}
```

## Constraints

1. **Plan mode focus**: Best used immediately after ExitPlanMode
2. **Deep context**: Read plan files and conversation for full context
3. **User confirmation**: Never record without user approval
4. **Sensitive filtering**: Always check mode and filter if public-safe
5. **No commit**: Design decisions commit with implementation, not separately
6. **Append only**: Never overwrite existing entries
