# claude-plugins

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://claude.ai/code)

A Claude Code plugin marketplace for development workflow automation.

## Table of Contents

- [Quick Start](#quick-start)
- [Plugins](#plugins)
  - [git](#git-plugin) - Git workflow automation
  - [react](#react-plugin) - React/Next.js development
  - [readme](#readme-plugin) - Documentation generation
  - [notion](#notion-plugin) - Notion TIL recording
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
```

---

## Plugins

### git plugin

Git workflow automation with smart commits, PRs, and CI monitoring.

| Command | Description |
|---------|-------------|
| `/git:commit` | Smart staging + conventional commit with gitmoji |
| `/git:commit-light` | Same as above, saves tokens |
| `/git:branch` | Create branch with `type/description` naming |
| `/git:pr` | Create PR with full code analysis |
| `/git:pr-light` | Create PR, saves tokens |
| `/git:ci` | Monitor GitHub Actions, analyze failures |
| `/git:init` | Setup husky + commitlint + gitmoji |

**Features:**
- Auto-detects staged/unstaged files
- Deep analysis reads actual file content
- Generates conventional commit messages with gitmoji
- Integrates with react, readme, notion plugins

📄 [Full documentation](./plugins/git/README.md)

---

### react plugin

React/Next.js code comment automation following CLAUDE.md conventions.

| Command | Description |
|---------|-------------|
| `/react:comment` | Add/reformat comments to React files |
| `/react:template` | Show comment template for file type |

**Supported files:** `*.tsx`, `*Context.tsx`, `*.service.ts`, `*.query.ts`, `*.dto.ts`, `*.utils.ts`

📄 [Full documentation](./plugins/react/README.md)

---

### readme plugin

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

### notion plugin

Record TIL (Today I Learned) entries to Notion database.

| Command | Description |
|---------|-------------|
| `/notion:til` | Record TIL to Notion (requires MCP) |

**Features:**
- Deep analysis of code changes
- Korean content with structured format (문제/해결/배운점)
- Auto tech stack detection from changed files
- Triggered after `/git:commit`

⚠️ Requires [Notion MCP setup](#notion-mcp-setup)

📄 [Full documentation](./plugins/notion/README.md)

---

## Workflow Examples

### Git Workflow

```bash
/git:branch          # Create feature/user-auth
# ... make changes ...
/git:commit          # ✨ feat: Add user authentication
/git:pr              # Create PR with analysis
/git:ci              # Monitor CI status
```

### Documentation Workflow

```bash
/readme:init         # Generate README.md

/git:pr
# → "📄 Update README?"
# → Yes → updates README

/git:commit
# → "📝 Record TIL to Notion?"
# → Yes → records TIL
```

### Plugin Integrations

| After | Prompt | Action |
|-------|--------|--------|
| `/git:commit` | "📝 Record TIL to Notion?" | `/notion:til` |
| `/git:commit` | "📝 React files detected..." | `/react:comment` |
| `/git:pr` | "📄 Update README?" | `/readme:update` |

---

## Requirements

| Requirement | For |
|-------------|-----|
| Git 2.0+ | All plugins |
| GitHub CLI (`gh`) | `/git:pr`, `/git:ci` |
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

📄 [MCP Guide](https://github.com/anthropics/claude-code/blob/main/docs/mcp.md)

---

## Contributing

Contributions welcome! Feel free to:
- Open issues for bugs or feature requests
- Submit PRs for improvements
- Suggest new commands or plugins

## License

MIT
