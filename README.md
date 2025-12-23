# claude-plugins

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://claude.ai/code)

A Claude Code plugin marketplace for development workflow automation.

## Table of Contents

- [Quick Start](#quick-start)
- [Plugins](#plugins)
  - [git](#git) - Git workflow automation
  - [react](#react) - React/Next.js development
  - [readme](#readme) - Documentation generation
  - [notion](#notion) - Notion workspace automation
  - [util](#util) - Utility tools
  - [devlog](#devlog) - Decision logging
- [Workflow Examples](#workflow-examples)
- [Requirements](#requirements)
- [Contributing](#contributing)

---

## Quick Start

```bash
# 1. Add marketplace
/plugin marketplace add minukHwang/claude-plugins

# 2. Install plugins you need
/plugin install git@minukHwang-plugins
/plugin install react@minukHwang-plugins
/plugin install readme@minukHwang-plugins
/plugin install notion@minukHwang-plugins
/plugin install util@minukHwang-plugins
/plugin install devlog@minukHwang-plugins
```

---

## Plugins

### git

Git workflow automation with smart commits, PRs, and CI monitoring.

| Command | Description |
|---------|-------------|
| `/git:commit` | Smart staging + conventional commit with gitmoji |
| `/git:commit-light` | Same as above, saves tokens |
| `/git:branch` | Create branch with `type/description` naming |
| `/git:pr` | Create PR with full code analysis |
| `/git:pr-light` | Create PR, saves tokens |
| `/git:ci` | Monitor GitHub Actions, analyze failures |
| `/git:init` | Full project init: git, .gitignore, hooks, first commit |

**Features:**
- Auto-detects staged/unstaged files
- Deep analysis reads actual file content
- Generates conventional commit messages with gitmoji
- Auto .gitignore generation (Node, Python, Go, Rust, Ruby, iOS, Android)
- Non-Node.js support (pre-commit framework)
- Integrates with react, readme, notion plugins

📄 [Full documentation](./plugins/git/README.md)

---

### react

React/Next.js code comment automation following CLAUDE.md conventions.

| Command | Description |
|---------|-------------|
| `/react:comment` | Add/reformat comments to React files |
| `/react:template` | Show comment template for file type |

| Skill | Description |
|-------|-------------|
| `react-conventions` | Auto-referenced for `.tsx`/`.ts` files |

**Supported files:** `*.tsx`, `*Context.tsx`, `*.service.ts`, `*.query.ts`, `*.dto.ts`, `*.utils.ts`

📄 [Full documentation](./plugins/react/README.md)

---

### readme

README auto-generation and updates based on project analysis.

| Command | Description |
|---------|-------------|
| `/readme:init` | Generate README based on project type |
| `/readme:update` | Update README with project changes |

**Features:**
- Detects project type (Frontend/Backend/Full-stack)
- Version-based status detection (< 1.0.0 = In Development)
- Placeholder support for early-stage projects
- Triggered after `/git:pr`

📄 [Full documentation](./plugins/readme/README.md)

---

### notion

Notion workspace automation tools.

| Command | Description |
|---------|-------------|
| `/notion:til` | Record TIL to Notion (requires MCP) |
| `/notion:blog` | Write detailed blog post (extends TIL) |

**Features:**
- Deep analysis of code changes
- **Multi-commit support**: Select multiple commits → 1 TIL
- **PR/MR support**: Analyze entire PR changes (GitHub & GitLab)
- Korean content with structured format (Problem/Solution/Lesson)
- Auto tech stack detection from changed files
- **TIL → Blog flow**: Expand TIL into detailed blog post
- **Web search**: Searches official docs and articles for blog
- Triggered after `/git:commit`

⚠️ Requires [Notion MCP setup](#notion-mcp-setup)

📄 [Full documentation](./plugins/notion/README.md)

---

### util

Utility tools for Claude Code.

| Skill | Description |
|-------|-------------|
| `search` | Current date awareness for web searches |

**Features:**
- Ensures Claude uses current year (not 2024) when searching
- Auto-triggers on: "검색해봐", "알아봐", "찾아봐", "분석해봐"
- Runs `date` before `WebSearch`/`WebFetch`

📄 [Full documentation](./plugins/util/README.md)

---

### devlog

Project decision and design logging with auto-trigger hooks.

| Command | Description |
|---------|-------------|
| `/devlog:init` | Initialize devlog files + optional hooks |
| `/devlog:log` | Record implementation decision to DEVLOG.md |
| `/devlog:plan` | Record design decision to PLANS.md |
| `/devlog:review` | Analyze and summarize decision history |

**Features:**
- Deep analysis of git diff and changed files
- Auto-extract problem/options/decision from conversation
- Git integration: public-safe or private mode
- Sensitive info filtering (API keys, passwords)
- Auto-trigger hooks (Stop + PreCompact)
- Context recovery for new sessions

📄 [Full documentation](./plugins/devlog/README.md)

---

## Workflow Examples

### Git Workflow

```bash
# New project setup
/git:init            # git init + .gitignore + husky + first commit

# Feature development
/git:branch          # Create feature/user-auth
# ... make changes ...
/git:commit          # ✨ feat: Add user authentication
/git:pr              # Create PR with analysis
/git:ci              # Monitor CI status
```

### React Workflow

```bash
/react:comment       # Add comments to React files
/react:template      # Show comment template

# Or after commit:
/git:commit
# → "📝 React files detected. Add comments?"
# → Yes
/react:comment
```

### README Workflow

```bash
/readme:init         # Generate new README
/readme:update       # Update existing README

# Or after PR:
/git:pr
# → "📄 Update README?"
# → Yes (README exists)
/readme:update
# → Yes (no README)
/readme:init
```

### Notion Workflow

```bash
/notion:til          # Record TIL to Notion
/notion:blog         # Write detailed blog post

# Or after commit:
/git:commit
# → "📝 Record TIL to Notion?"
# → Yes
/notion:til
# → "📝 Expand to blog post?"
# → Yes
/notion:blog
```

### Devlog Workflow

```bash
# Initialize in new project
/devlog:init         # Set up files + optional hooks

# After Plan mode design
# → Auto-suggested via Stop hook
/devlog:plan         # Record design decision

# After implementation decisions
# → Auto-suggested via Stop hook
/devlog:log          # Record implementation decision

# Before compact or new session
# → Auto-reminded via PreCompact hook
/devlog:review       # Review decision history
```

---

## Requirements

| Requirement | For |
|-------------|-----|
| Git 2.0+ | All plugins |
| GitHub CLI (`gh`) | `/git:pr`, `/git:ci`, `/notion:til` (GitHub) |
| GitLab CLI (`glab`) | `/notion:til` (GitLab PR/MR) |
| Node.js | `/git:init` |
| macOS or Linux | All plugins |
| Notion MCP | `/notion:til` |

### Notion MCP Setup

```bash
# 1. Add Notion MCP
claude mcp add notion -- npx -y @anthropic-ai/notion-mcp

# 2. Create integration at https://www.notion.so/my-integrations

# 3. Restart Claude Code
```

📄 [MCP Guide](https://docs.anthropic.com/en/docs/claude-code/mcp)

---

## Contributing

Contributions welcome! Feel free to:
- Open issues for bugs or feature requests
- Submit PRs for improvements
- Suggest new commands or plugins

## License

MIT
