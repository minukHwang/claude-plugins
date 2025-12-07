# claude-plugins

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://claude.ai/code)

A Claude Code plugin marketplace for Git workflow automation. Streamline your commit, PR, branch, and CI workflows with AI-powered commands.

## Features

- **Smart Commits** - Auto-stage files, generate conventional commit messages with gitmoji
- **PR Automation** - Create comprehensive PRs with deep code analysis
- **Branch Management** - Create branches with consistent naming conventions
- **CI Monitoring** - Check GitHub Actions status and analyze failures
- **Project Setup** - Initialize husky, commitlint, and gitmoji in one command

## Quick Start

```bash
# 1. Add marketplace
/plugin marketplace add minukHwang/claude-plugins

# 2. Install git plugin
/plugin install git@minukHwang-plugins

# 3. Start using!
/git:commit
```

## Commands Overview

### git plugin

| Command | Description |
|---------|-------------|
| `/git:commit` | Smart staging + conventional commit with deep analysis |
| `/git:commit-light` | Same as above, but saves tokens (git diff only) |
| `/git:branch` | Create branch with `type/description` naming |
| `/git:pr` | Create PR with full code analysis |
| `/git:pr-light` | Create PR with minimal analysis (saves tokens) |
| `/git:ci` | Monitor GitHub Actions, analyze failures |
| `/git:init` | Setup husky + commitlint + gitmoji |

## Workflow Example

```bash
# 1. Create a feature branch
/git:branch
# → Select: feature
# → Enter: user-authentication
# ✓ Branch created: feature/user-authentication

# 2. Make your changes, then commit
/git:commit
# → Auto-detects staged/unstaged files
# → Generates: ✨ feat: Add user authentication with JWT
# ✓ Commit created: a1b2c3d

# 3. Create a pull request
/git:pr
# → Analyzes all commits and changes
# → Creates PR with summary, key changes, review points
# ✓ PR created: https://github.com/user/repo/pull/123

# 4. Monitor CI status
/git:ci
# → Shows pass/fail status for all checks
# → If failed: analyzes logs and suggests fixes
```

## Smart Features

### Intelligent Staging (`/git:commit`)

| Situation | Behavior |
|-----------|----------|
| All files staged | Proceeds without asking |
| Partial staging | Confirms: "Commit only these files?" |
| Nothing staged | Asks what to add (all / path / describe) |

### Cascading Selection (`/git:branch`)

For options with 4+ choices, uses cascading menus:

```
Round 1: feature / bugfix / hotfix / Other →
Round 2: release / docs / refactor / Other →
Round 3: test / chore / [custom input]
```

### Deep vs Light Mode

| Feature | Deep (`/git:commit`) | Light (`/git:commit-light`) |
|---------|:--------------------:|:---------------------------:|
| Git diff analysis | ✅ | ✅ |
| File content reading | ✅ | ❌ |
| Conversation context | ✅ | ❌ |
| Token usage | Higher | Lower |

Use **light mode** when you want faster responses and lower token usage.

## Commit Types & Gitmoji

| Type | Emoji | Description |
|------|:-----:|-------------|
| feat | ✨ | New feature |
| fix | 🐛 | Bug fix |
| docs | 📄 | Documentation |
| style | 🎨 | Code style/formatting |
| refactor | 📦 | Code restructuring |
| perf | 🚀 | Performance improvement |
| test | 🚨 | Tests |
| build | 🔨 | Build/dependencies |
| ci | 🔧 | CI/CD configuration |
| chore | 📝 | Maintenance |
| revert | 🗑 | Revert changes |
| init | 🎉 | Initial setup |

## Requirements

- **Git** 2.0+
- **GitHub CLI** (`gh`) - for PR and CI commands
- **Node.js** - for `/git:init` (husky setup)
- **macOS or Linux**

## Project Setup (`/git:init`)

Automatically sets up commit tooling for your project:

```bash
/git:init
```

**What it does:**
1. Detects package manager (pnpm/npm/yarn)
2. Installs husky, commitlint, gitmoji
3. Creates git hooks (prepare-commit-msg, commit-msg, pre-commit)
4. Auto-detects linting tools for pre-commit

**Non-Node.js projects?** Shows alternatives:
- Python → `pre-commit`
- Go → `golangci-lint`
- Rust → `cargo-husky`
- Ruby → `overcommit`

## Plugin Structure

```
claude-plugins/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── git/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       │   ├── commit.md
│       │   ├── commit-light.md
│       │   ├── branch.md
│       │   ├── pr.md
│       │   ├── pr-light.md
│       │   ├── ci.md
│       │   └── init.md
│       └── README.md
└── README.md
```

## Contributing

Contributions are welcome! Feel free to:
- Open issues for bugs or feature requests
- Submit PRs for improvements
- Suggest new commands or plugins

## License

MIT
