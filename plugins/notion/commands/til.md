---
description: Record TIL (Today I Learned) to Notion database
---

# Record TIL to Notion

Analyze work done and create a TIL entry in Notion database (Korean).

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
4. Run /notion:til again

MCP Guide: https://docs.anthropic.com/en/docs/claude-code/mcp
```
→ Stop

## Step 1: Analysis Target Selection (Standalone Mode)

**Ask user:**
"What would you like to record?"

| Option | Description |
|--------|-------------|
| 1 | Analyze recent commits |
| 2 | Enter specific commit hash |
| 3 | Describe manually |

※ When triggered from `/git:commit`: Auto-analyze the commit just made (reuse conversation context)

### Option 1: Recent Commit

```bash
git log -5 --oneline
```

**Ask user:**
"How many commits to analyze?"

| Option | Description |
|--------|-------------|
| 1 | Last 1 commit |
| 2 | Last 3 commits |
| 3 | Last 5 commits |

### Option 2: Specific Commit

User enters commit hash

### Option 3: Manual Description

User describes the work done

## Step 2: Deep Analysis (`/git:commit` Level)

| Step | Analysis |
|------|----------|
| 2.1 Commit Analysis | `git log -1 --format="%B"` (commit message) |
| 2.2 Diff Analysis | `git diff HEAD~1` (changes) |
| 2.3 Changed Files | `git diff HEAD~1 --name-only` |
| 2.4 **File Content** | **Read changed files with Read tool** |
| 2.5 Context Analysis | Conversation context (reuse if from git integration) |
| 2.6 Tech Stack Extract | Determine tech from file extensions/imports |

### Tech Stack Detection Rules

| Indicator | Tech |
|-----------|------|
| `.tsx` files changed | React |
| `useQuery` import | React Query |
| `axios` import | Axios |
| Tailwind classes in code | Tailwind CSS |
| `.css` files | CSS |
| `prisma` usage | Prisma |
| `next/` imports | Next.js |
| `.java` files | Java |
| `@Controller` annotation | Spring Boot |

**Important:** Only include tech actually used in THIS commit, not entire project stack.

### Area Detection

| Indicator | Area |
|-----------|------|
| `.tsx`, `.jsx`, `.vue`, `.css` | Frontend |
| `.java`, `.go`, `.py` (server) | Backend |
| `Dockerfile`, `.yml` (CI) | DevOps |
| `terraform`, `k8s` | Infra |
| Both Frontend + Backend | Full-stack |

### Analysis Output

- TIL content (Problem/Solution/Learned)
- Area (Frontend/Backend/etc.)
- Tech stack (only for this commit)
- Commit type (feat/fix/docs/etc.)

## Step 3: Find or Create TIL Database

### 3.1 Search for Existing DB

```
mcp__notion__notion-search
query: "[Claude] TIL"
```

### 3.2 Handle Search Results

#### Case A: 1 result found
→ Use that database

#### Case B: Multiple results found

**Ask user:**
"Multiple TIL databases found. Which one to use?"

Show list of found databases → User selects one

#### Case C: No results found

**Ask user:**
"Where should the TIL database be created?"

| Option | Description |
|--------|-------------|
| 1 | Search and select a page |
| 2 | Enter page URL/name directly |
| 3 | Create at workspace root |

**Option 1: Search and Select**
```
mcp__notion__notion-search
query: [user's search term]
```
→ Show results → User selects parent page
→ Create DB under selected page

**Option 2: Direct Input**
User enters page URL or name
```
mcp__notion__notion-fetch
id: [user input]
```
→ Verify page exists
→ Create DB under that page

**Option 3: Workspace Root**
```
mcp__notion__notion-create-database
(no parent specified)
```

### 3.3 Create Database (if needed)

```
mcp__notion__notion-create-database

title: [Claude] TIL
properties:
  - 제목 (title)
  - 날짜 (date)
  - 타입 (select): feat, fix, docs, style, refactor, perf, test, build, ci, chore
  - 영역 (select): Frontend, Backend, DevOps, Infra, Full-stack
  - 기술 스택 (multi_select): React, Next.js, TypeScript, Spring Boot, etc.
  - 프로젝트 (url)
  - 커밋 (url)
```

## Step 4: Get Git Repository Info

```bash
# Get repo URL
git remote get-url origin

# Get current commit hash
git rev-parse HEAD

# Parse owner/repo from URL
# https://github.com/{owner}/{repo}.git → extract owner, repo
```

## Step 5: Create TIL Entry (Korean)

### Page Properties

| Property | Value |
|----------|-------|
| 제목 | Work summary (Korean, based on analysis) |
| 날짜 | Today's date |
| 타입 | feat/fix/docs/etc. (from commit) |
| 영역 | Frontend/Backend/etc. (based on analysis) |
| 기술 스택 | Only tech used in this commit |
| 프로젝트 | `https://github.com/{owner}/{repo}` |
| 커밋 | `https://github.com/{owner}/{repo}/commit/{hash}` |

### Page Content (Korean)

```markdown
## 🔍 문제

[분석 기반으로 어떤 문제/이슈가 있었는지 작성]

## 💡 해결

[어떻게 해결했는지, 핵심 코드 예시 포함]

## 📚 배운점

[새로 알게 된 것, 시행착오, 팁 등]
```

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
    기술 스택: "React, TypeScript",
    프로젝트: "https://github.com/...",
    커밋: "https://github.com/.../commit/..."
  },
  content: "## 🔍 문제\n..."
}]
```

## Output Format

### On Success:
```
✓ TIL recorded!

Title: [TIL title]
Area: [Frontend/Backend/etc.]
Tech Stack: [React, TypeScript, ...]

View in Notion: [page URL]
```

### On Failure:
```
✗ Failed to record TIL: <error message>
```

## Constraints

1. **Korean content**: TIL content in Korean
2. **Accurate tech stack**: Only include tech actually used in this work
3. **Deep analysis**: Read actual file content, don't just rely on diffs
4. **Git context reuse**: When from `/git:commit`, reuse analysis to save tokens
5. **DB naming**: Always use `[Claude] TIL` for database name
6. **MCP required**: Cannot work without Notion MCP connection
