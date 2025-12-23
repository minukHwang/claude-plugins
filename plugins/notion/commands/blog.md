---
description: Write detailed technical blog post to Notion (extends TIL with web search)
---

# Write Technical Blog to Notion

Analyze work done and create a detailed blog post in Notion database (Korean content).
Extends TIL with deeper analysis, web search for references, and structured format.

## Step 0: MCP Connection Check

Try calling Notion MCP tool:
```
mcp__notion__notion-get-self
```

If MCP not available:
```
⚠️ Notion MCP is not connected.

Setup:
1. claude mcp add notion -- npx -y @anthropic-ai/notion-mcp
2. Set Notion Integration token (https://www.notion.so/my-integrations)
3. Restart Claude Code
4. Run /notion:blog again

MCP Guide: https://docs.anthropic.com/en/docs/claude-code/mcp
```
→ Stop

## Step 1: Analysis Target Selection

**Ask user (AskUserQuestion):**
"무엇을 블로그로 작성할까요?"

| Option | Description |
|--------|-------------|
| 최신 커밋 1개 | 현재 브랜치 HEAD 커밋 분석 |
| 특정 커밋 선택 | 최근 커밋 목록에서 선택 (다중 선택 가능) |
| PR/MR 단위 | PR/MR 목록에서 선택 |
| TIL에서 확장 | 기존 TIL을 블로그로 확장 |
| 직접 입력 | 수동으로 작업 내용 설명 |

### Option: TIL에서 확장

```
mcp__notion__notion-search
query: "[CLAUDE] TIL"
```

Show recent TIL list → User selects one → Fetch TIL page → Step 2

### Other Options
→ Same as TIL plugin

## Step 2: Deep Analysis

### 2.1 Commit/PR Analysis
```bash
git log -1 --format="%B" {hash}  # commit message
git diff {hash}~1..{hash}        # changes (or PR range)
git diff {hash}~1..{hash} --name-only  # changed files
```

### 2.2 File Content Analysis
**Read changed files with Read tool** to understand:
- What was modified and why
- Technical decisions involved
- Patterns, new components, or modified logic

### 2.3 Context Analysis
If there's relevant conversation history:
- What task was the user working on?
- What problem were they solving?
- Any specific feature names or terminology mentioned?

### 2.4 Tech Stack Detection
| Indicator | Tech |
|-----------|------|
| `.tsx` files | React |
| `useQuery` import | React Query |
| `next/` imports | Next.js |
| Tailwind classes | Tailwind CSS |
| `prisma` usage | Prisma |

**Important:** Only include tech actually used in THIS commit/PR.

### 2.5 TIL DB Lookup (Additional)

Check for related TIL (use config or search):

```bash
# Check user config first
cat ~/.claude/notion.json 2>/dev/null
# If til.id exists, use that DB
# Otherwise search:
```

```
mcp__notion__notion-search
query: "[CLAUDE] TIL"
```

Search TIL DB for same commit/PR reference:
- Check if 참조 field contains same commit hash or PR number
- If found, fetch TIL page for reference

**If TIL exists:**
- Use TIL content as draft/skeleton
- Expand with more details

**If TIL not found:**
- Create blog from code analysis only

## Step 3: Web Search for References (Additional)

Search for tech stack documentation:

```
WebSearch
query: "{tech stack} official documentation 2025"
```

**Search targets:**
- Official documentation
- Related best practices
- Similar problem solutions

**Collect:**
- 2-3 official doc links
- 1-2 related articles
- GitHub Issues (if applicable)

## Step 4: Fetch Blog Writing Guide (Additional)

Fetch the blog writing guide:

```
mcp__notion__notion-fetch
id: "2d18429a-1c2a-813f-9887-d3b3408c44d3"
```

**Follow the guide structure exactly:**
1. 배경 (Context)
2. 문제 정의 (Problem)
3. 기술적 선택지 (Options) ⭐ - Key differentiator
4. 해결 및 구현 (Implementation)
5. 성과 및 회고 (Impact & Learning)
6. 참고 자료

**Important:** Write detailed content for each section as specified in the guide.

## Step 4.5: Devlog Enhancement Prompt (Optional)

After content analysis, before creating blog entry:

**Ask user (AskUserQuestion):**
"📋 Enhance with devlog context?"

| Option | Description |
|--------|-------------|
| Yes | Find related entries from devlog |
| No | Proceed without devlog |

### If "Yes":

Run devlog lookup (cascade filtering):

1. **Match by commit hash**: Current commit(s) in DEVLOG.md/PLANS.md **Commit** field
2. **Match by Related Files**: Compare changed files with entry's **Related Files**
3. **Fallback by branch**: Match current branch, show recent 3 entries, ask user to confirm

If found: Use entry context to enhance blog content (especially Options/Decision sections).
If not found: "No related devlog found" → proceed.

**Reference:** DEVLOG.md + PLANS.md

## Step 5: Find or Create Blog Database

### 5.1 Check User Config

```bash
cat ~/.claude/notion.json 2>/dev/null
```

If `blog.id` exists in config → Use existing DB, skip to Step 6

### 5.2 Search Notion for Existing DB (if not in config)

```
mcp__notion__notion-search
query: "[CLAUDE] BLOG"
filter: { "property": "object", "value": "database" }
```

#### Case A: Found
- Extract database ID
- Save to `~/.claude/notion.json` (see 5.5)
- Display: "Found existing [CLAUDE] BLOG database"
- Use that database

#### Case B: Not Found

**Ask user (AskUserQuestion):**
"[CLAUDE] BLOG database not found. What would you like to do?"

| Option | Description |
|--------|-------------|
| Create new | Create new [CLAUDE] BLOG database |
| Skip | Don't record blog |
| Enter ID | Provide existing database ID manually |

**If "Create new":**
→ Go to 5.4

**If "Skip":**
→ Exit with message: "Blog recording skipped. Run /notion:blog again when ready."

**If "Enter ID":**
User provides database ID → Verify via `mcp__notion__notion-fetch` → Save to config

### 5.3 기존 DB 마이그레이션 (if needed)

기존 `[Claude] Blog` DB가 있는 경우:
- 속성 구조 확인 후 필요시 마이그레이션

### 5.4 Create Database (if not found)

```
mcp__notion__notion-create-database

parent: { "type": "workspace", "workspace": true }
title: [CLAUDE] BLOG
properties:
  - 제목 (title)
  - 날짜 (date)
  - 타입 (select): feat, fix, docs, style, refactor, perf, test, build, ci, chore
  - 영역 (select): Frontend, Backend, DevOps, Infra, Full-stack
  - 기술 스택 (multi_select)
  - 프로젝트 (rich_text): Repo name with link
  - 참조 (rich_text): Commit/PR/TIL links
  - 상태 (select): 작성중, 완료, 발행됨
```

### 5.5 Save to User Config

Create `~/.claude` directory if not exists:
```bash
mkdir -p ~/.claude
```

Read existing config or create new:
```bash
cat ~/.claude/notion.json 2>/dev/null || echo '{}'
```

Update with BLOG DB info and write:

```json
{
  "blog": {
    "id": "{database_id}",
    "name": "[CLAUDE] BLOG"
  }
}
```

**Note:** Merge with existing config (preserve todo, til fields if present)

## Step 6: Create Blog Entry

### Page Properties

| Property | Value |
|----------|-------|
| 제목 | Descriptive title (Korean) |
| 날짜 | Today's date |
| 타입 | feat/fix/docs/etc. |
| 영역 | Frontend/Backend/etc. |
| 기술 스택 | Tech used in this work |
| 프로젝트 | `[{project}](repo_url)` |
| 참조 | Commit/PR links + TIL link (if exists) |
| 상태 | 작성중 |

### Page Content

**⚠️ Important: Blog is much more detailed than TIL!**

Follow the blog writing guide (fetched in Step 4) exactly:
- Each section must meet minimum length requirements
- Use comparison tables for Options section
- Use Before/After tables for quantitative results
- Include code snippets with comments
- Add web search results to 참고 자료

### Create Page

```
mcp__notion__notion-create-pages
parent: { data_source_id: [DB data source ID] }
pages: [{
  properties: {
    제목: "...",
    date:날짜:start: "YYYY-MM-DD",
    date:날짜:is_datetime: 0,
    타입: "feat",
    영역: "Frontend",
    기술 스택: "[\"React\", \"TypeScript\"]",
    프로젝트: "[{project}](repo_url)",
    참조: "[{hash}]({commit_url}) | [TIL]({til_url})",
    상태: "작성중"
  },
  content: "## 1. 배경 (Context)\n..."
}]
```

## Output Format

### On Success:
```
✓ Blog post created!

Title: [Blog title]
Area: [Frontend/Backend/etc.]
Tech Stack: [React, TypeScript, ...]
Status: 작성중

View in Notion: [page URL]

💡 Tip: 작성 완료 후 상태를 "완료"로 변경하세요.
```

### On Failure:
```
✗ Failed to create blog post: <error message>
```

## Constraints

1. **Korean content**: Blog content in Korean
2. **Detailed writing**: Much more detailed than TIL
3. **Web search required**: Always search for references
4. **Guide-based structure**: Follow the blog writing guide exactly
5. **TIL reference**: Check and use related TIL if exists
6. **Accurate tech stack**: Only include tech actually used
7. **MCP required**: Cannot work without Notion MCP connection
8. **DB naming**: Always use `[CLAUDE] BLOG` for database name
9. **User-level config**: Store DB ID in `~/.claude/notion.json`, not project-level
