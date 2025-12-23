---
name: devlog-reminder
description: MANDATORY after completing design decisions or implementation work - suggests /devlog:plan for design or /devlog:log for implementation to record decisions
---

<MANDATORY>
After completing ANY of these tasks, you MUST suggest devlog:

1. **Design decisions** → Suggest `/devlog:plan`
2. **Implementation work** → Suggest `/devlog:log`

This is not optional. Every significant decision must be recorded.
</MANDATORY>

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

```
[Work completion message]

💡 Record this decision/implementation?
- `/devlog:plan` - Record design decision
- `/devlog:log` - Record implementation

📦 If context is getting long, run `/compact` after recording
```

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
