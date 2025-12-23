---
description: Analyze and summarize decision history from devlog files
---

# Review Devlog

Analyze DEVLOG.md and PLANS.md to summarize decision history and patterns.

## Step 0: Check Devlog Exists

Check for devlog files:
1. Project root: `DEVLOG.md`, `PLANS.md`
2. Private location: `.claude/devlog/DEVLOG.md`, `.claude/devlog/PLANS.md`

If not found:
```
⚠️ Devlog not found.

Run /devlog:init first to set up devlog files.
```
→ Stop

## Step 1: Read Files

Read both files with Read tool:
- DEVLOG.md (implementation decisions)
- PLANS.md (design decisions)

## Step 2: Confirm Review Scope

**Ask user (AskUserQuestion):**
"What would you like to review?"

| Option | Description |
|--------|-------------|
| Full summary | Complete overview of all decisions |
| Recent N | Last N decisions only |
| Search keyword | Find decisions by keyword |
| Context recovery | Quick summary for new session |

### If Recent N:
Ask: "How many recent decisions?" (default: 5)

### If Search keyword:
Ask: "Enter search keyword:"

## Step 3: Analyze and Output

### 3.1 Parse Entries

Parse each entry by `###` delimiter:
- Date/time
- Branch
- Problem/Goal
- Options considered
- Decision made
- Outcome

### 3.2 Generate Summary

**Output Format:**

```markdown
## 📊 Devlog Summary

### Overview
- Total decisions: {count}
- Design decisions: {plan_count}
- Implementation decisions: {log_count}
- Date range: {earliest} - {latest}

---

### Recent Decisions ({n})

#### Implementation (DEVLOG.md)
| Date | Branch | Problem | Decision |
|------|--------|---------|----------|
| {date} | {branch} | {problem} | {decision} |
| ... | ... | ... | ... |

#### Design (PLANS.md)
| Date | Branch | Goal | Approach |
|------|--------|------|----------|
| {date} | {branch} | {goal} | {approach} |
| ... | ... | ... | ... |

---

### Common Trade-off Patterns
{Analyze recurring trade-offs}

- **Performance vs Complexity**: {count} occurrences
  - Examples: {brief list}
- **Scalability vs Simplicity**: {count} occurrences
  - Examples: {brief list}

---

### Technology Choices
{Analyze frequently chosen technologies}

| Technology | Times Chosen | Context |
|------------|--------------|---------|
| {tech1} | {count} | {typical use case} |
| {tech2} | {count} | {typical use case} |

---

### Decision Patterns
{Identify patterns in decision-making}

- Most common problem types: {list}
- Preferred approaches: {list}
- Recurring concerns: {list}
```

### 3.3 Context Recovery Mode

If "Context recovery" selected, provide condensed summary:

```markdown
## 🔄 Quick Context Recovery

### Current State
- Last activity: {date}
- Active branch: {branch}
- Recent focus: {summary}

### Key Decisions to Remember
1. {decision1} - {reason}
2. {decision2} - {reason}
3. {decision3} - {reason}

### Open Items
- {open question 1}
- {open question 2}

### Recommended Next Steps
- {suggestion based on history}
```

## Output

### On Success:
Display the formatted summary directly.

### On Failure:
```
✗ Failed to analyze devlog: {error message}
```

## Use Cases

1. **New session start**: Quick context recovery
2. **Before similar decision**: Reference past choices
3. **Team onboarding**: Understand project decision history
4. **Retrospective**: Review patterns and learnings

## Constraints

1. **Read-only**: This command only reads, never modifies
2. **Parse carefully**: Handle malformed entries gracefully
3. **Summarize intelligently**: Focus on actionable insights
4. **Privacy aware**: Don't expose redacted content details
