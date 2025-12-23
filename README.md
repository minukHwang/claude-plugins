# claude-plugins

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://claude.ai/code)

A Claude Code plugin marketplace for development workflow automation.

## Table of Contents

- [Quick Start](#quick-start)
- [Plugins](#plugins)
  - [workflow](#workflow) - Project workflow configuration (NEW)
  - [jira](#jira) - Jira issue management (NEW)
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
/plugin install workflow@minukHwang-plugins  # NEW: Project config
/plugin install jira@minukHwang-plugins      # NEW: Jira integration
/plugin install git@minukHwang-plugins
/plugin install react@minukHwang-plugins
/plugin install readme@minukHwang-plugins
/plugin install notion@minukHwang-plugins
/plugin install util@minukHwang-plugins
/plugin install devlog@minukHwang-plugins
```

---

## Plugins

### workflow

Project workflow configuration for Git strategy, Jira integration, and Notion sync.

| Command | Description |
|---------|-------------|
| `/workflow:init` | Initialize workflow configuration |
| `/workflow:config` | Modify existing workflow settings |

| Skill | Description |
|-------|-------------|
| `workflow-context` | Auto-load workflow settings |
| `one-pr-one-issue` | 1 PR = 1 Issue workflow guide (v1.2.0) |

**Features:**
- Git strategy selection (github-flow, git-flow, trunk-based)
- Jira project connection with issue key in branches/commits
- Notion TODO database integration with **Epic field** (v1.1.0)
- **Type field**: Epic/Story/Task/Bug/Todo select (v1.2.0)
- **CLAUDE.md rules**: Auto-add 1 PR = 1 Issue workflow (v1.2.0)
- **Two-ID system**: Data Source ID + Page ID for Notion DBs (v1.1.0)
- Merge method configuration (squash, merge, rebase)
- Creates `.claude/workflow.json` for other plugins to use

⚠️ Requires [Atlassian MCP](#atlassian-mcp-setup) for Jira integration

📄 [Full documentation](./plugins/workflow/README.md)

---

### jira

Jira issue management and Git integration automation.

| Command | Description |
|---------|-------------|
| `/jira:backlog` | View project backlog issues |
| `/jira:create` | Create a new Jira issue |
| `/jira:start` | Start working on issue (creates branch) |
| `/jira:done` | Complete issue (creates PR, updates status) |
| `/jira:view` | View issue details |
| `/jira:sync` | Bidirectional sync: Jira ↔ Notion TODO (v1.3.0) |

**Features:**
- Full Jira workflow: create → start → done
- **Epic selection**: Link issues to parent Epics (v1.1.0)
- **Due Date & Auto-assign**: Optional due date, auto-assign to self (v1.1.0)
- **Start Date**: Optional at creation, auto-set on start (v1.3.0)
- Auto branch creation with issue key (via `/git:branch`)
- Automatic status transitions with **Start Date** (v1.1.0)
- Notion TODO sync (create, update status, add links)
- **Type field**: Epic/Story/Task/Bug synced to Notion (v1.3.0)
- **Epic field**: Rich text link to Epic (v1.3.0)
- **Bidirectional sync**: Jira ↔ Notion with Done propagation (v1.3.0)
- Seamless integration with git plugin

⚠️ Requires [Atlassian MCP](#atlassian-mcp-setup) and `/workflow:init`

📄 [Full documentation](./plugins/jira/README.md)

---

### git

Git workflow automation with smart commits, PRs, CI monitoring, and Jira integration.

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
- **Jira integration** (v1.4.0): Issue key in branch/commit/PR
- **Notion PR update**: Saves PR link to TODO `[#42](url)` (v1.5.0)
- **Multi-issue warning**: Warns on 1 PR with multiple issues (v1.5.0)

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
| `react-conventions` | MANDATORY - Auto-applied for `.tsx`/`.ts` files |

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
- **Two-ID system**: Data Source ID + Page ID (v1.6.0)
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

Project decision and design logging with Confluence sync.

| Command | Description |
|---------|-------------|
| `/devlog:init` | Initialize devlog files + optional hooks |
| `/devlog:log` | Record implementation decision to DEVLOG.md |
| `/devlog:plan` | Record design decision to PLANS.md |
| `/devlog:review` | Analyze and summarize decision history |
| `/devlog:confluence` | Sync devlog to Confluence (NEW) |

| Skill | Description |
|-------|-------------|
| `devlog-reminder` | MANDATORY - Auto-suggests devlog after significant work |

**Features:**
- Deep analysis of git diff and changed files
- Auto-extract problem/options/decision from conversation
- Git integration: public-safe or private mode
- Sensitive info filtering (API keys, passwords)
- Skill-based auto-reminder (devlog-reminder)
- Context recovery for new sessions
- Integration with `/git:commit`, `/git:pr`, `/notion:til`, `/notion:blog`
- **Confluence sync** (v1.2.0): Sync entries to Confluence page

📄 [Full documentation](./plugins/devlog/README.md)

---

## Workflow Examples

### Jira Workflow (NEW)

```bash
# 1. Initialize workflow config
/workflow:init       # Set up Git strategy + Jira + Notion

# 2. Create and start issue
/jira:create         # Create issue + Notion TODO
/jira:start CP-1     # Create branch + update status

# 3. Work and commit
# ... make changes ...
/git:commit          # ✨ feat: Add feature [CP-1]

# 4. Complete
/jira:done           # Create PR + update status + sync Notion
```

### Git Workflow

```bash
# New project setup
/git:init            # git init + .gitignore + husky + first commit

# Feature development
/git:branch          # Create feature/user-auth (with optional Jira link)
# ... make changes ...
/git:commit          # ✨ feat: Add user authentication [CP-1]
/git:pr              # Create PR with analysis + Jira link
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
/devlog:init         # Set up files + install devlog-reminder skill

# After Plan mode design
# → Auto-suggested via devlog-reminder skill
/devlog:plan         # Record design decision

# After implementation decisions
# → Auto-suggested via devlog-reminder skill
/devlog:log          # Record implementation decision

# Before compact or new session
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
| Atlassian MCP | `/workflow:init`, `/jira:*`, `/devlog:confluence` |
| Notion MCP | `/notion:til`, `/jira:create` (TODO sync) |

### Atlassian MCP Setup

```bash
# 1. Add Atlassian MCP (requires authentication)
claude mcp add atlassian -- npx -y @anthropic-ai/atlassian-mcp

# 2. Follow authentication prompts for Jira/Confluence access

# 3. Restart Claude Code
```

Required permissions:
- Jira: Read/write issues, transitions
- Confluence: Read/write pages (for devlog sync)

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
