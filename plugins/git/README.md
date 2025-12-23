# git

GitHub commit, PR, branch, and CI automation plugin for Claude Code with optional Jira integration.

## Version

1.4.0

## Jira Integration (New in 1.4.0)

When `.claude/workflow.json` is configured with Jira settings, git commands automatically integrate with Jira:

| Command | Jira Feature |
|---------|--------------|
| `/git:branch` | Link issue, include key in branch name, update status |
| `/git:commit` | Append `[ISSUE-KEY]` to commit message |
| `/git:pr` | Add Jira issue link in PR body |

**Requires:** `/workflow:init` with Jira enabled

**Example with Jira:**
```
/git:branch
# Link to CP-1? → Yes
# Creates: feature/CP-1-add-authentication
# Updates Jira status: To Do → In Progress

/git:commit
# ✨ feat: Add login functionality [CP-1]

/git:pr
# Includes: ## Jira Issue
# [CP-1](https://site.atlassian.net/browse/CP-1) - Add authentication
```

## Features

### `/git:commit` - Create Commit

Analyzes staged changes and generates a conventional commit message.

**Features:**
- **Integrated git add** - If no staged changes, prompts to add files
- **Auto-detects commitlint config** - If found, merges with plugin rules
- Analyzes `git diff --staged` to understand changes
- Generates conventional commit format (`✨ type: description`)
- Automatically chooses single-line or bullet-point format based on complexity
- Always includes gitmoji prefix (works with or without commitlint)

**Smart staging:**
| Situation | Behavior |
|-----------|----------|
| All staged | Proceed without asking |
| Partial staged | Confirm: "Proceed with only staged files?" |
| Nothing staged | Ask what to add (all / path / describe) |

**Auto-detection:**
```
commitlint found → Uses their types/limits + our emoji format
commitlint not found → Uses defaults (emoji + 100 chars + standard types)
```

**Usage:**
```bash
/git:commit
# If no staged changes, will prompt to add files
```

**Devlog integration:**
- After commit, updates DEVLOG.md entries with commit hash (if Related Files match)
- Offers "📋 Enhance with devlog context?" before TIL prompt

**Example output:**
```
✓ Commit created: a1b2c3d
  ✨ feat: Add emotion calendar view with monthly navigation
```

---

### `/git:commit-light` - Create Commit (Light Mode)

Same as `/git:commit` but **saves tokens** by skipping deep analysis.

| Feature | `/git:commit` | `/git:commit-light` |
|---------|:-------------:|:-------------------:|
| Integrated git add | ✅ | ✅ |
| Git diff analysis | ✅ | ✅ |
| Commitlint detection | ✅ | ✅ |
| File content reading | ✅ | ❌ |
| Conversation context | ✅ | ❌ |

**Usage:**
```bash
/git:commit-light
```

---

### `/git:branch` - Create Branch

Creates a new branch with proper naming convention and checks it out.

**Branch naming formats:**
- Personal: `<type>/<description>`
- Team: `<accountName>/<type>/<description>`

**Branch types (cascading selection):**
| Type | Description |
|------|-------------|
| feature | New feature |
| bugfix | Bug fix |
| hotfix | Urgent production fix |
| release | Release preparation |
| docs | Documentation |
| refactor | Code restructuring |
| test | Testing changes |
| chore | Maintenance |
| Custom | Enter your own type |

**Usage:**
```bash
/git:branch
```

**Example output:**
```
✓ Branch created and checked out: feature/emotion-calendar
```

---

### `/git:pr` - Create Pull Request

Analyzes all changes and creates a comprehensive PR using `gh` CLI.

**Analysis depth:**
1. **Commit messages** - Understanding intent
2. **Git diff** - What code changed
3. **File content** - Context and impact
4. **Conversation context** - User's goals from session

**PR format:**
- Title: `<emoji> <type>: <description>` (same gitmoji as commits)
- Body: Overview, Key Changes, Technical Details, Review Points, Testing checklist

**Usage:**
```bash
/git:pr
```

**Devlog integration:**
- Offers "📋 Enhance with devlog context?" to reference DEVLOG.md + PLANS.md

**Example output:**
```
✓ Branch pushed to origin
✓ Pull Request created successfully!

Title: ✨ feat: Add emotion calendar with monthly navigation
URL: https://github.com/user/repo/pull/123
```

---

### `/git:pr-light` - Create Pull Request (Light Mode)

Same as `/git:pr` but **saves tokens** by skipping deep analysis.

| Feature | `/git:pr` | `/git:pr-light` |
|---------|:---------:|:---------------:|
| Commit messages | ✅ | ✅ |
| Git diff analysis | ✅ | ✅ |
| File content reading | ✅ | ❌ |
| Conversation context | ✅ | ❌ |

**Usage:**
```bash
/git:pr-light
```

---

### `/git:ci` - Monitor CI Status

Checks GitHub Actions CI status and analyzes failures.

**Features:**
- Shows CI check status (pass/fail/pending)
- Analyzes failed logs automatically
- Provides actionable fix suggestions
- Offers to re-run failed checks
- Watch mode for pending checks

**Usage:**
```bash
/git:ci
```

**Example output (passing):**
```
✅ All CI checks passed!

| Check | Status | Duration |
|-------|--------|----------|
| build | ✓ Pass | 2m 30s |
| test  | ✓ Pass | 3m 15s |
| lint  | ✓ Pass | 45s |

Ready to merge! 🎉
```

**Example output (failing):**
```
❌ CI checks failed

| Check | Status | Duration |
|-------|--------|----------|
| build | ✓ Pass | 2m 30s |
| test  | ✗ Fail | 1m 45s |

📍 Error: src/utils/calc.test.ts:25
   Expected: 10, Received: 9

💡 The calculation in `add()` might be off by one.
```

---

### `/git:init` - Initialize Git Project

Complete git project initialization from repository setup to first commit.

**What it does:**

| Step | Feature |
|------|---------|
| 0 | **Git repository** - `git init` + remote (connect or create via `gh`) |
| 1 | **Auto .gitignore** - Detect project type, generate template |
| 2 | **Project type check** - Node.js → husky, Others → pre-commit/manual |
| 3-10 | **Husky setup** - commitlint, gitmoji hooks (Node.js only) |
| 11 | **Initial commit** - Launch `/git:commit` with `init` type |

**.gitignore templates:**
Node.js, Python, Go, Rust, Ruby, iOS, Android, Generic

**Remote options:**
| Option | Description |
|--------|-------------|
| Existing URL | Connect to existing GitHub repo |
| Create new | Create repo via `gh repo create` |
| Skip | Local only |

**Non-Node.js support:**
| Option | Description |
|--------|-------------|
| pre-commit | Install pre-commit framework (Python) |
| Manual | Create hooks in `.git/hooks/` |
| Skip | No hooks |

**Usage:**
```bash
/git:init
```

**Workflow connection:**
```
/git:init → /git:commit → /git:pr → /notion:til
```

## Requirements

- Git 2.0+
- GitHub CLI (`gh`) installed and authenticated
- macOS or Linux
- Node.js + pnpm/npm/yarn (for `/git:init`)

## Installation

```bash
# Add marketplace
/plugin marketplace add minukHwang/claude-plugins

# Install plugin
/plugin install git@minukHwang-plugins
```

## Workflow Example

```bash
# 1. Create a new branch
/git:branch
# Select: feature
# Enter: user-authentication

# 2. Make your changes...

# 3. Commit (will prompt to add files if needed)
/git:commit

# 4. Create PR when ready
/git:pr

# 5. Monitor CI status
/git:ci
# If failed: analyze errors and fix
```

## Commit Types

| Type | Description | Gitmoji |
|------|-------------|---------|
| feat | New feature | ✨ |
| fix | Bug fix | 🐛 |
| docs | Documentation | 📄 |
| style | Code style/format | 🎨 |
| refactor | Code restructuring | 📦 |
| perf | Performance | 🚀 |
| test | Tests | 🚨 |
| build | Build/dependencies | 🔨 |
| ci | CI/CD | 🔧 |
| chore | Maintenance | 📝 |
| revert | Revert commit | 🗑 |
| init | Initial setup | 🎉 |
| release | Version release | 🔖 |

## License

MIT
