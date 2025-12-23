# Devlog Plugin

Project decision and design logging with auto-trigger hooks.

## Purpose

Record implementation decisions and design choices to prevent context loss across sessions. Especially useful when:
- Claude Code compacts context
- Starting a new session on the same project
- Reviewing past architectural decisions
- Onboarding team members

## Commands

| Command | Description |
|---------|-------------|
| `/devlog:init` | Initialize devlog files and optional global hooks |
| `/devlog:log` | Record implementation decision to DEVLOG.md |
| `/devlog:plan` | Record design decision to PLANS.md |
| `/devlog:review` | Analyze and summarize decision history |

## Quick Start

```bash
# Initialize devlog in your project
/devlog:init

# After making implementation decisions
/devlog:log

# After Plan mode design sessions
/devlog:plan

# Review decision history
/devlog:review
```

## Features

### Deep Analysis
- Analyzes git diff and changed files
- Extracts context from conversation
- Auto-detects problem/options/decision

### Git Integration
- Public-safe mode: Store in project root, commit with code
- Private mode: Store in `.claude/devlog/`, gitignored

### Sensitive Info Filtering
In public-safe mode, automatically redacts:
- API keys and tokens
- Passwords
- Internal URLs
- IP addresses

### Auto-trigger Hooks
Optional global hooks for automatic suggestions:
- **Stop hook**: Suggests `/devlog:plan` after Plan mode, `/devlog:log` after decisions
- **PreCompact hook**: Reminds to log before context compaction

## File Structure

### DEVLOG.md (Implementation Decisions)
```markdown
### 2025-01-15 14:30 ### feature/user-service

**Commit**: abc1234 - "feat: add caching layer"

**Context**
Working on user service API...

**Problem**
API responses are slow...

**Options Considered**
| Option | Pros | Cons |
|--------|------|------|
| Redis | ... | ... |
| In-memory | ... | ... |

**Decision Rationale**
- Multi-instance environment...

**Outcome/Lessons**
- TTL configuration was critical...
```

### PLANS.md (Design Decisions)
```markdown
### 2025-01-15 14:30 ### feature/user-auth

**Design Goal**
Implement secure user authentication...

**Approaches Considered**
| Approach | Pros | Cons |
|----------|------|------|
| JWT + Redis | ... | ... |
| Session-based | ... | ... |

**Architecture Decisions**
- Access token: JWT...
- Refresh token: HTTP-only cookie...

**Trade-offs Accepted**
- Complexity vs Scalability...
```

## Hook Configuration

Add to `~/.claude/settings.json` for all projects:

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

## Version

1.0.0
