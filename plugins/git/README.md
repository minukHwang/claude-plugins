# git

GitHub commit, PR, and branch automation plugin for Claude Code.

## Features

### `/git:commit` - Create Commit

Analyzes staged changes and generates a conventional commit message.

**Features:**
- **Auto-detects commitlint config** - If found, merges with plugin rules
- Analyzes `git diff --staged` to understand changes
- Generates conventional commit format (`✨ type: description`)
- Automatically chooses single-line or bullet-point format based on complexity
- Always includes gitmoji prefix (works with or without commitlint)

**Auto-detection:**
```
commitlint found → Uses their types/limits + our emoji format
commitlint not found → Uses defaults (emoji + 100 chars + standard types)
```

**Usage:**
```bash
git add <files>
/git:commit
```

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
| Git diff analysis | ✅ | ✅ |
| Commitlint detection | ✅ | ✅ |
| File content reading | ✅ | ❌ |
| Conversation context | ✅ | ❌ |

**Usage:**
```bash
git add <files>
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

## Requirements

- Git 2.0+
- GitHub CLI (`gh`) installed and authenticated
- macOS or Linux

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

# 3. Stage and commit
git add .
/git:commit

# 4. Create PR when ready
/git:pr
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

## License

MIT
