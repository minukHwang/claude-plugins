# Notion Plugin

Notion workspace automation tools.

## Commands

| Command | Description |
|---------|-------------|
| `/notion:til` | Record TIL (Today I Learned) to Notion database |

## Requirements

⚠️ This plugin requires **Notion MCP** to be configured.

### Setup

```bash
# 1. Add Notion MCP
claude mcp add notion -- npx -y @anthropic-ai/notion-mcp

# 2. Create Notion Integration
# Go to: https://www.notion.so/my-integrations
# Create a new integration and copy the token

# 3. Set the token (follow MCP setup guide)

# 4. Restart Claude Code

# 5. Ready to use!
/notion:til
```

For detailed MCP setup: [Claude Code MCP Guide](https://docs.anthropic.com/en/docs/claude-code/mcp)

## /notion:til

Records a TIL (Today I Learned) entry to a Notion database.

### Workflow

```bash
/notion:til
# → "What would you like to record?"
# → 1. Analyze recent commits
# → "How many commits to analyze?"
# → 1. Last 1 commit
# → Analyzing changes...
# ✓ TIL recorded to Notion!
```

### Features

- **Deep Analysis**: Reads actual file content, not just diffs
- **Auto Tech Stack**: Extracts tech stack from changed files only
- **Korean Content**: TIL content written in Korean
- **Git Links**: Includes project and commit URLs

### TIL Database

**Database Name:** `[Claude] TIL`

| Property | Type | Description |
|----------|------|-------------|
| 제목 | Title | Work summary (Korean) |
| 날짜 | Date | Today's date |
| 타입 | Select | feat/fix/docs/refactor/etc. |
| 영역 | Select | Frontend/Backend/DevOps/Infra/Full-stack |
| 기술 스택 | Multi-select | Tech used in THIS work only |
| 프로젝트 | URL | GitHub repository link |
| 커밋 | URL | Commit or PR link |

### TIL Page Content (Korean)

```markdown
## 🔍 문제
[What problem/issue existed]

## 💡 해결
[How it was solved, with code examples]

## 📚 배운점
[What was learned, tips, insights]
```

### Deep Analysis

Same depth as `/git:commit`:

| Step | Analysis |
|------|----------|
| Commit Analysis | `git log -1 --format="%B"` |
| Diff Analysis | `git diff HEAD~1` |
| Changed Files | `git diff --name-only` |
| **File Content** | **Read tool for actual content** |
| Tech Stack | Extract from file extensions/imports |

### Tech Stack Detection

Only includes tech actually used in the commit:

| Indicator | Tech |
|-----------|------|
| `.tsx` files changed | React |
| `useQuery` import | React Query |
| `axios` usage | Axios |
| Tailwind classes | Tailwind CSS |
| `prisma` usage | Prisma |
| `next/` imports | Next.js |

### Area Detection

| Indicator | Area |
|-----------|------|
| `.tsx`, `.jsx`, `.vue`, `.css` | Frontend |
| `.java`, `.go`, `.py` (server) | Backend |
| `Dockerfile`, `.yml` (CI) | DevOps |
| `terraform`, `k8s` | Infra |
| Both Frontend + Backend | Full-stack |

### Database Location

First run behavior:

1. Searches for existing `[Claude] TIL` database
2. If multiple found → asks which to use
3. If none found → asks where to create:

| Option | Description |
|--------|-------------|
| 1 | Search and select a page |
| 2 | Enter page URL directly |
| 3 | Create at workspace root |

## git:commit Integration

After committing with `/git:commit`:

```
✓ Commit created: a1b2c3d

📝 Record TIL to Notion?
1. Yes
2. No
```

Selecting "Yes":
- Runs `/notion:til`
- Auto-analyzes the commit just made
- Reuses analysis context (saves tokens)

## Examples

### Standalone Usage

```bash
/notion:til
# → "What would you like to record?"
# → 1. Analyze recent commits
# → Analyzes commit and creates TIL
# ✓ TIL recorded!
```

### After Commit (Integrated)

```bash
/git:commit
# → ✓ Commit created: a1b2c3d
# → "📝 Record TIL to Notion?"
# → Yes
# → Creates TIL using existing analysis
# ✓ TIL recorded! (saves tokens)
```

### Specific Commit

```bash
/notion:til
# → "What would you like to record?"
# → 2. Enter specific commit hash
# → Enter: abc123
# → Analyzes that commit
# ✓ TIL recorded!
```

## License

MIT
