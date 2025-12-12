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

**Ask user (AskUserQuestion):**
"무엇을 기록할까요?"

| Option | Description |
|--------|-------------|
| 최신 커밋 1개 | 현재 브랜치 HEAD 커밋 분석 |
| 특정 커밋 선택 | 최근 커밋 목록에서 선택 (다중 선택 가능) |
| PR/MR 단위 | PR/MR 목록에서 선택 |
| 직접 입력 | 수동으로 작업 내용 설명 |

※ When triggered from `/git:commit`: Auto-analyze the commit just made (reuse conversation context)

### Option 1: 최신 커밋 1개

→ Step 2-A로 이동 (단일 커밋 분석)

### Option 2: 특정 커밋 선택

```bash
git log --oneline --relative-date -10
```

**Ask user (AskUserQuestion, multiSelect: true):**
"분석할 커밋을 선택하세요"

| Option | Description |
|--------|-------------|
| `{hash} {message}` | `{relative_date}` |
| `{hash} {message}` | `{relative_date}` |
| `{hash} {message}` | `{relative_date}` |
| 직접 입력 | 커밋 해시 직접 입력 |

※ 최근 3개 커밋 + 직접 입력 옵션 = 4개 (AskUserQuestion 제한)

→ 1개 선택: Step 2-A로 이동
→ 여러 개 선택: Step 2-B로 이동 (다중 커밋 분석)

### Option 3: PR/MR 단위

먼저 호스트 감지 (Step 4 로직 선행):
```bash
git remote get-url origin
```

**GitHub:**
```bash
gh pr list --limit 3 --json number,title,headRefName,baseRefName
```

**GitLab:**
```bash
glab mr list --per-page 3
```

**Ask user (AskUserQuestion):**
"분석할 PR/MR을 선택하세요"

| Option | Description |
|--------|-------------|
| `#123 {title}` | `{head} → {base}` |
| `#456 {title}` | `{head} → {base}` |
| `#789 {title}` | `{head} → {base}` |
| 직접 입력 | PR/MR 번호 직접 입력 |

→ Step 2-C로 이동 (PR 분석)

### Option 4: 직접 입력

User describes the work done

→ Step 2-D로 이동 (수동 분석)

## Step 2: Deep Analysis

### Step 2-A: 단일 커밋 분석

| Step | Analysis |
|------|----------|
| 2.1 Commit Analysis | `git log -1 --format="%B" {hash}` (commit message) |
| 2.2 Diff Analysis | `git diff {hash}~1..{hash}` (changes) |
| 2.3 Changed Files | `git diff {hash}~1..{hash} --name-only` |
| 2.4 **File Content** | **Read changed files with Read tool** |
| 2.5 Context Analysis | Conversation context (reuse if from git integration) |
| 2.6 Tech Stack Extract | Determine tech from file extensions/imports |

### Step 2-B: 다중 커밋 분석

| Step | Analysis |
|------|----------|
| 2.1 Commit Analysis | `git log {oldest}..{newest} --pretty=format:"%h %s%n%b"` (선택된 모든 커밋 메시지) |
| 2.2 Diff Analysis | `git diff {oldest}~1..{newest}` (전체 범위) |
| 2.3 Changed Files | `git diff {oldest}~1..{newest} --name-only` |
| 2.4 **File Content** | **Read changed files with Read tool** |
| 2.5 Context Analysis | Conversation context |
| 2.6 Tech Stack Extract | Determine tech from file extensions/imports |

→ 여러 커밋 → **TIL 1개**로 합침

### Step 2-C: PR/MR 분석

먼저 PR 정보 조회:

**GitHub:**
```bash
gh pr view {number} --json baseRefName,headRefName
```

**GitLab:**
```bash
glab mr view {number} --output json
```

| Step | Analysis |
|------|----------|
| 2.1 Commit Analysis | `git log origin/{base}..HEAD --pretty=format:"%h %s%n%b"` |
| 2.2 Diff Analysis | `git diff origin/{base}..HEAD` |
| 2.3 Changed Files | `git diff origin/{base}..HEAD --name-only` |
| 2.4 **File Content** | **Read changed files with Read tool** |
| 2.5 Context Analysis | Conversation context |
| 2.6 Tech Stack Extract | Determine tech from file extensions/imports |

### Step 2-D: 수동 분석

사용자가 직접 작업 내용 설명 → 해당 내용 기반으로 TIL 작성

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

**Ask user (AskUserQuestion):**
"Multiple TIL databases found. Which one to use?"

Show list of found databases as options → User selects one

#### Case C: No results found

**Ask user (AskUserQuestion):**
"Where should the TIL database be created?"

| Option | Description |
|--------|-------------|
| Search and select | Search for a page and select it |
| Enter directly | Enter page URL or name directly |
| Workspace root | Create at workspace root level |

**If "Search and select":**
```
mcp__notion__notion-search
query: [user's search term]
```
→ Show results → User selects parent page
→ Create DB under selected page

**If "Enter directly":**
User enters page URL or name
```
mcp__notion__notion-fetch
id: [user input]
```
→ Verify page exists
→ Create DB under that page

**If "Workspace root":**
```
mcp__notion__notion-create-database
(no parent specified)
```

### 3.3 기존 DB 마이그레이션 (if needed)

기존 `[Claude] TIL` DB가 있고 "커밋" 속성이 url 타입인 경우:

1. 기존 데이터 조회
2. "참조" 속성 추가 (rich_text)
3. 기존 "커밋" URL 값을 `[{hash}]({url})` 형식으로 변환하여 "참조"에 저장
4. "커밋" 속성 삭제

### 3.4 Create Database (if needed)

```
mcp__notion__notion-create-database

title: [Claude] TIL
properties:
  - 제목 (title)
  - 날짜 (date)
  - 타입 (select): feat, fix, docs, style, refactor, perf, test, build, ci, chore
  - 영역 (select): Frontend, Backend, DevOps, Infra, Full-stack
  - 기술 스택 (multi_select): React, Next.js, TypeScript, Spring Boot, etc.
  - 프로젝트 (rich_text): Repo name with link
  - 참조 (rich_text): Commit/PR links in markdown format
```

## Step 4: Get Git Repository Info

```bash
# Get repo URL
git remote get-url origin

# Get current commit hash
git rev-parse HEAD
```

### Parse Remote URL

| Pattern | Host Type |
|---------|-----------|
| `github.com` 포함 | GitHub |
| `gitlab` 포함 (gitlab.com, gitlab.company.com 등) | GitLab |
| 그 외 | 호스트 그대로 사용 (GitHub 형식 가정) |

### URL Parsing Rules

**SSH format:** `git@{host}:{namespace}/{project}.git`
**HTTPS format:** `https://{host}/{namespace}/{project}.git`

→ `{host}`, `{namespace}`, `{project}` 추출

## Step 5: Create TIL Entry (Korean)

### Page Properties

| Property | Value |
|----------|-------|
| 제목 | Work summary (Korean, based on analysis) |
| 날짜 | Today's date |
| 타입 | feat/fix/docs/etc. (from commit) |
| 영역 | Frontend/Backend/etc. (based on analysis) |
| 기술 스택 | Only tech used in this commit |
| 프로젝트 | `[{project}](https://{host}/{namespace}/{project})` (repo name as link) |
| 참조 | 분석 타입에 따라 다름 (아래 참고) |

### 참조 필드 형식 (rich_text, markdown)

| 분석 타입 | 형식 |
|-----------|------|
| 커밋 1개 | `[{short_hash}]({commit_url})` |
| 커밋 여러 개 | `[{hash1}]({url1}) [{hash2}]({url2}) ...` |
| PR/MR | `[#{number}]({pr_url})` |

**URL 형식:**
- GitHub commit: `https://{host}/{namespace}/{project}/commit/{hash}`
- GitLab commit: `https://{host}/{namespace}/{project}/-/commit/{hash}`
- GitHub PR: `https://{host}/{namespace}/{project}/pull/{number}`
- GitLab MR: `https://{host}/{namespace}/{project}/-/merge_requests/{number}`

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
    프로젝트: "[{project}](https://{host}/{namespace}/{project})",
    참조: "[{short_hash}]({commit_url})"  // or "[#{number}]({pr_url})" for PR/MR
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
