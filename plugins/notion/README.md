# Notion Plugin

Notion workspace automation tools.

## Version

1.6.0

## Commands

| Command | Description |
|---------|-------------|
| `/notion:til` | Record TIL (Today I Learned) to Notion database |
| `/notion:blog` | Write detailed technical blog post to Notion (extends TIL) |

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
# → "무엇을 기록할까요?"
# → 1. 최신 커밋 1개
# → 2. 특정 커밋 선택 (다중 선택 가능)
# → 3. PR/MR 단위
# → 4. 직접 입력
# → Analyzing changes...
# ✓ TIL recorded to Notion!
```

### Features

- **Deep Analysis**: Reads actual file content, not just diffs
- **Auto Tech Stack**: Extracts tech stack from changed files only
- **Korean Content**: TIL content written in Korean
- **Multi-commit Support**: Select multiple commits → merged into 1 TIL
- **PR/MR Support**: Analyze entire PR/MR changes (GitHub & GitLab)
- **Git Links**: Repo name + commit/PR links (markdown format)
- **Devlog Integration**: "📋 Enhance with devlog context?" prompt (DEVLOG.md + PLANS.md)

### TIL Database

**Database Name:** `[CLAUDE] TIL`
**Config Location:** `~/.claude/notion.json` (user-level)

| Property | Type | Description |
|----------|------|-------------|
| 제목 | Title | Work summary (Korean) |
| 날짜 | Date | Today's date |
| 타입 | Select | feat/fix/docs/refactor/etc. |
| 영역 | Select | Frontend/Backend/DevOps/Infra/Full-stack |
| 기술 스택 | Multi-select | Tech used in THIS work only |
| 프로젝트 | Text | Repo name as clickable link |
| 참조 | Text | Commit/PR links in markdown: `[abc123](url)` or `[#123](url)` |

### TIL Page Content (Korean)

```markdown
## 📋 배경
- 상황: Context of the work
- 문제 인식: Why this work was needed
- 목표: What we wanted to achieve

## 🔍 문제
### ⚠️ 1. [Problem Title]
- 현상: What went wrong
- 원인 분석: Why it happened

## 💡 해결
### 고려한 선택지
- A 방법: ...
- B 방법: ...

### ⭐ 대안 비교
| 선택지 | 장점 | 단점 |
|--------|------|------|
| A 방법 | ... | ... |
| **B 방법 (선택)** | ... | - |

### 선택한 방법과 이유
[Why this approach was chosen]

### 구현 코드
[Code examples]

## ✅ 결과
[What was achieved]

## 📚 배운점
### 🔧 기술적 배움
### 🧠 설계적 배움

## 🤔 더 알아보기
[AI suggestions for further learning]

## 🔗 관련 링크
[Reference links]
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

First run behavior (TIL and BLOG DBs):

1. Check `~/.claude/notion.json` for existing DB ID
2. If not found → Search Notion for `[CLAUDE] TIL` or `[CLAUDE] BLOG`
3. If not found → Ask user:

| Option | Description |
|--------|-------------|
| Create new | Create new database at workspace root |
| Skip | Don't record, try again later |
| Enter ID | Provide existing database ID manually |

DB IDs are saved to `~/.claude/notion.json` (user-level, shared across projects).

**ID Types (v1.6.0):**
- `id`: Data Source ID (collection://) - for creating pages
- `pageId`: Database Page ID - for schema updates

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

### Latest Commit

```bash
/notion:til
# → "무엇을 기록할까요?"
# → 1. 최신 커밋 1개
# → Analyzes HEAD commit
# ✓ TIL recorded!
```

### Multiple Commits

```bash
/notion:til
# → "무엇을 기록할까요?"
# → 2. 특정 커밋 선택
# → "분석할 커밋을 선택하세요" (multiSelect)
# → Select: abc123, def456
# → Merges into 1 TIL
# ✓ TIL recorded!
```

### PR/MR Analysis

```bash
/notion:til
# → "무엇을 기록할까요?"
# → 3. PR/MR 단위
# → "분석할 PR/MR을 선택하세요"
# → #123 Add login feature
# → Analyzes entire PR diff
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

## /notion:blog

Writes a detailed technical blog post to Notion. Extends TIL with deeper analysis, web search, and structured format.

### Workflow

```bash
/notion:blog
# → "무엇을 블로그로 작성할까요?"
# → 1. 최신 커밋 1개
# → 2. 특정 커밋 선택 (다중 선택 가능)
# → 3. PR/MR 단위
# → 4. TIL에서 확장
# → 5. 직접 입력
# → Deep analysis + Web search...
# ✓ Blog post created!
```

### TIL vs Blog

| Aspect | TIL | Blog |
|--------|-----|------|
| Depth | Minimal | Rich & detailed |
| Structure | 배경/문제/해결/결과/배운점 | 배경/문제/선택지⭐/구현/성과 |
| Web Search | ❌ | ✅ |
| TIL Reference | - | ✅ |
| Database | `[CLAUDE] TIL` | `[CLAUDE] BLOG` |

### Features

- **TIL Extension**: Expand existing TIL into detailed blog post
- **Web Search**: Searches official docs and related articles
- **Blog Writing Guide**: Follows structured guide (Notion page)
- **Options Section**: Detailed comparison of technical alternatives
- **Before/After Tables**: Quantitative results comparison

### Blog Database

**Database Name:** `[CLAUDE] BLOG`
**Config Location:** `~/.claude/notion.json` (user-level)

| Property | Type | Description |
|----------|------|-------------|
| 제목 | Title | Descriptive title (Korean) |
| 날짜 | Date | Today's date |
| 타입 | Select | feat/fix/docs/refactor/etc. |
| 영역 | Select | Frontend/Backend/DevOps/Infra/Full-stack |
| 기술 스택 | Multi-select | Tech used in this work |
| 프로젝트 | Text | Repo name as clickable link |
| 참조 | Text | Commit/PR/TIL links in markdown |
| 상태 | Select | 작성중/완료/발행됨 |

### Blog Page Content

Follows the blog writing guide structure:

```markdown
## 1. 배경 (Context)
- 상황, 문제 인식, 환경

## 2. 문제 정의 (Problem)
- 현상, 디버깅 과정, 근본 원인

## 3. 기술적 선택지 (Options) ⭐
- 비교 테이블
- 각 Option 상세 (동작 원리, 검토 과정, 결론)
- 선택 이유

## 4. 해결 및 구현 (Implementation)
- 아키텍처/설계
- 구현 코드
- UX 디테일 (optional)
- 트러블슈팅 요약

## 5. 성과 및 회고 (Impact & Learning)
- 정량적 성과 (Before/After table)
- 정성적 성과 (applicable items only)
- 배운 점 (기술/설계/UX/비즈니스)
- 회고

## 🔗 참고 자료
- Web search collected links
```

### TIL → Blog Flow

After creating a TIL, you'll be prompted:

```
✓ TIL recorded!

📝 블로그로 확장할까요?
1. 예 - 지금 바로 블로그로 확장
2. 나중에 - TIL만 기록하고 종료
```

Selecting "예" will start `/notion:blog` with the TIL content as reference.

## License

MIT
