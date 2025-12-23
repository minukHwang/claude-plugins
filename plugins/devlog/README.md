# Devlog Plugin

Project decision and design logging with Confluence sync.

## Version

1.3.0

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
| `/devlog:confluence` | Sync devlog entries to Confluence (New in 1.2.0) |

## Skills

| Skill | Description |
|-------|-------------|
| `devlog-reminder` | MANDATORY - Auto-suggests `/devlog:plan` or `/devlog:log` after completing design or implementation work |

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

# Sync to Confluence (requires workflow.json)
/devlog:confluence
```

## Confluence Integration (New in 1.2.0)

When `.claude/workflow.json` has Confluence enabled, devlog commands offer automatic sync:

**Setup:**
```bash
/workflow:init
# Enable Confluence during setup
```

**Auto-sync prompts:**
- `/devlog:log` → "Also record to Confluence?"
- `/devlog:plan` → "Also record to Confluence?"

**Manual sync:**
```bash
/devlog:confluence
# Syncs DEVLOG.md and PLANS.md to "[Project] Development Log" page
```

**Features:**
- Auto-creates Confluence page if not exists
- Sync latest entry or full history
- Preserves local and remote versions

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

### Auto-trigger Reminders
- **Skill-based**: `devlog-reminder` skill automatically suggests logging after design/implementation work
- **PreCompact hook**: Optional hook reminds to log before context compaction

## File Structure

### DEVLOG.md (Implementation Decisions)
```markdown
### 2025-01-15 14:30 ### feature/user-service

**Commit**: abc1234 - "feat: add caching layer"

**Related Files**
- `src/cache/redis.service.ts`
- `src/user/user.controller.ts`

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

**Note:**
- **Commit** field starts empty; updated automatically after `/git:commit`
- **Related Files** enables accurate matching when referenced by other commands

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

## Integration with Other Commands

Other commands can reference devlog for enhanced context:

| Command | Devlog Reference | Prompt |
|---------|------------------|--------|
| `/git:commit` | DEVLOG.md | "📋 Enhance with devlog context?" |
| `/git:pr` | DEVLOG.md + PLANS.md | "📋 Enhance with devlog context?" |
| `/notion:til` | DEVLOG.md + PLANS.md | "📋 Enhance with devlog context?" |
| `/notion:blog` | DEVLOG.md + PLANS.md | "📋 Enhance with devlog context?" |

### Lookup Flow (Cascade Filtering)

1. **Match by commit hash**: Find entries with matching **Commit** field
2. **Match by Related Files**: Compare changed files with entry's **Related Files**
3. **Fallback by branch**: Match current branch, show recent 3 entries
4. If not found: "No related devlog found"

### Auto Hash Update

After `/git:commit`, the commit hash is automatically added to matching DEVLOG.md entries based on **Related Files**.

## Hook Configuration

### Skill-based Reminder (Recommended)

The `devlog-reminder` skill is installed to `~/.claude/skills/` during `/devlog:init`. It automatically suggests devlog commands after completing significant work.

### Optional PreCompact Hook

Add to `~/.claude/settings.json` for reminder before context compaction:

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

