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
query: "[Claude] TIL"
```

Show recent TIL list → User selects one → Fetch TIL page → Step 2

### Other Options
→ Same as TIL plugin

## Step 2: Deep Analysis

### 2.1 Code Analysis (Same as TIL)

| Step | Analysis |
|------|----------|
| Commit/PR Analysis | git log, git diff |
| Changed Files | Read with Read tool |
| Context Analysis | Conversation context |
| Tech Stack Extract | From file extensions/imports |

### 2.2 TIL DB Lookup (Additional)

Check for related TIL:

```
mcp__notion__notion-search
query: "[Claude] TIL"
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

## Step 5: Find or Create Blog Database

### 5.1 Search for Existing DB

```
mcp__notion__notion-search
query: "[Claude] Blog"
```

### 5.2 Handle Search Results

- **1 result:** Use that database
- **Multiple results:** Ask user to select
- **No results:** Ask user where to create

### 5.3 Create Database (if needed)

```
mcp__notion__notion-create-database

title: [Claude] Blog
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
