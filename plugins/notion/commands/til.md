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

### 3.1 Check User Config

```bash
cat ~/.claude/notion.json 2>/dev/null
```

If `til.id` exists in config → Use existing DB, skip to Step 4

### 3.2 Search Notion for Existing DB (if not in config)

```
mcp__notion__notion-search
query: "[CLAUDE] TIL"
filter: { "property": "object", "value": "database" }
```

#### Case A: Found
- Extract database ID
- Save to `~/.claude/notion.json` (see 3.5)
- Display: "Found existing [CLAUDE] TIL database"
- Use that database

#### Case B: Not Found

**Ask user (AskUserQuestion):**
"[CLAUDE] TIL database not found. What would you like to do?"

| Option | Description |
|--------|-------------|
| Create new | Create new [CLAUDE] TIL database |
| Skip | Don't record TIL |
| Enter ID | Provide existing database ID manually |

**If "Create new":**
→ Go to 3.4

**If "Skip":**
→ Exit with message: "TIL recording skipped. Run /notion:til again when ready."

**If "Enter ID":**
User provides database ID → Verify via `mcp__notion__notion-fetch` → Save to config

### 3.3 기존 DB 마이그레이션 (if needed)

기존 `[Claude] TIL` DB가 있고 "커밋" 속성이 url 타입인 경우:

1. 기존 데이터 조회
2. "참조" 속성 추가 (rich_text)
3. 기존 "커밋" URL 값을 `[{hash}]({url})` 형식으로 변환하여 "참조"에 저장
4. "커밋" 속성 삭제

### 3.4 Create Database (if not found)

```
mcp__notion__notion-create-database

parent: { "type": "workspace", "workspace": true }
title: [CLAUDE] TIL
properties:
  - 제목 (title)
  - 날짜 (date)
  - 타입 (select): feat, fix, docs, style, refactor, perf, test, build, ci, chore
  - 영역 (select): Frontend, Backend, DevOps, Infra, Full-stack
  - 기술 스택 (multi_select): React, Next.js, TypeScript, Spring Boot, etc.
  - 프로젝트 (rich_text): Repo name with link
  - 참조 (rich_text): Commit/PR links in markdown format
```

### 3.5 Save to User Config

Create `~/.claude` directory if not exists:
```bash
mkdir -p ~/.claude
```

Read existing config or create new:
```bash
cat ~/.claude/notion.json 2>/dev/null || echo '{}'
```

Update with TIL DB info and write:

```json
{
  "til": {
    "id": "{data_source_id}",
    "pageId": "{database_page_id}",
    "name": "[CLAUDE] TIL"
  }
}
```

**ID Types:**
- `id`: Data Source ID (collection://) - for creating pages
- `pageId`: Database Page ID - for schema updates

**Note:** Merge with existing config (preserve todo, blog fields if present)

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

## Step 4.5: File Reference Prompt (Optional)

After content analysis, before creating TIL entry:

**Ask user (AskUserQuestion):**
"📁 Reference other files?"

| Option | Description |
|--------|-------------|
| Yes | Enter file path |
| No | Skip |

### If "Yes":

User enters file path → Read file with Read tool → Use content to enhance TIL content (especially Problem/Solution sections).

Example paths:
- `docs/plans/PLAN_feature.md` - Plan file
- `DECISIONS.md` - Decision log
- Any relevant documentation file

## Step 4.6: Validate & Add Tech Stack Options (CRITICAL)

**⚠️ 반드시 TIL 생성 전에 수행! 이 단계를 건너뛰면 에러 발생**

### 4.6.1 Get Current DB Options

```
mcp__notion__notion-fetch
id: {til.pageId from ~/.claude/notion.json}
```

→ 응답에서 `기술 스택` 옵션 목록 추출

### 4.6.2 Compare with Detected Tech

```
감지된 기술: [React, Bash, Lighthouse, TypeScript]
DB 옵션: [React, TypeScript, Next.js, ...]

없는 기술: [Bash, Lighthouse]
```

### 4.6.3 Add Missing Options (if any)

없는 기술이 있으면 DB에 추가:

```
mcp__notion__notion-update-database
database_id: {til.pageId}
properties: {
  "기술 스택": {
    "type": "multi_select",
    "multi_select": {
      "options": [
        ...기존 옵션들,
        {"name": "Bash", "color": "default"},
        {"name": "Lighthouse", "color": "default"}
      ]
    }
  }
}
```

**Note:** 기존 옵션 + 새 옵션 모두 포함해야 함 (덮어쓰기됨)

### 4.6.4 Continue to Step 5

옵션 추가 완료 후 TIL 생성 진행

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

**⚠️ 중요: 각 섹션을 충분히 상세하게 작성할 것!**
간단한 한 줄이 아니라, 맥락과 구체적인 내용을 포함해야 함.

---

#### 기본 구조

```markdown
# 📋 배경
# 🔍 문제
# 💡 해결
# ✅ 결과
# 📚 배운점
# 🤔 더 알아보기
# 🔗 관련 링크
```

---

#### 📋 배경

**목적**: 작업의 컨텍스트를 제공

**포함할 내용:**
- **상황**: 어떤 프로젝트/기능에서 작업했는지
- **문제 인식**: 왜 이 작업이 필요했는지
- **목표**: 최종적으로 달성하려는 것

**예시:**
```markdown
## 상황
ZeroGravity의 프로필 페이지에는 **감정 캘린더**가 있다.
- **Desktop**: 월별 캘린더 + 사이드 드로워
- **Mobile**: 주별 캘린더 + 하단 상세 영역

## 문제 인식
날짜를 전환할 때 **로딩 상태 제어**가 필요했다.

## 목표
**데이터가 없을 때 깜빡임 없이 부드럽게 전환되는 UX**를 만드는 것.
```

---

#### 🔍 문제

**목적**: 구체적인 문제 상황을 명확히 기술

> 💡 문제가 여러 개면 1, 2, 3... n까지 모두 나열. 각 문제마다 현상/원인 분석을 각각 작성.

**포함할 내용:**
- **현상**: 어떤 문제가 발생했는지 (구체적인 증상, 에러 메시지)
- **원인 분석**: 왜 이 문제가 발생했는지 (근본 원인, 코드/로직 문제점)
- **재현 방법**: 문제를 재현하는 단계 (있다면)

**구조:**
```markdown
# 🔍 문제
## ⚠️ 1. [문제 제목]
### 현상
[무엇이 잘못되었는지 - 2-3줄]

### 원인 분석
[왜 이런 일이 발생했는지 - 2-3줄]

## ⚠️ 2. [문제 제목]
...

## ⚠️ n. [문제 제목]
...
```

---

#### 💡 해결

**목적**: 문제를 어떻게 해결했는지 상세히 기술

> 💡 해결책이 여러 개면 (Desktop/Mobile 등) 각각 작성. 복잡한 해결은 **단계별 접근**으로 구체화.

**포함할 내용 (순서대로):**
1. **고려한 선택지**: 어떤 방법들을 고려했는지 (2-3개)
2. **⭐ 대안 비교**: 각 선택지의 장단점 테이블 (**매우 중요!**)
3. **선택한 방법과 이유**: 왜 이 방법을 선택했는지 (2-3줄)
4. **핵심 아이디어**: 선택한 방법의 구체적인 접근법 (3-5줄)
5. **단계별 접근**: 복잡한 경우 단계별로 설명 (선택)
6. **구현 코드**: 실제 적용한 코드 (5-15줄)

> ⚠️ **⭐ 대안 비교가 매우 중요한 이유:**
> 어떤 선택지가 있었고, 왜 이 방법을 선택했는지 비교하면:
> - 나중에 돌아봤을 때 "왜 이렇게 했지?" 이해 가능
> - 비슷한 상황에서 의사결정 기준이 됨
> - 트레이드오프를 명확히 인식할 수 있음

**구조:**
```markdown
# 💡 해결
## 🖥️ [해결책 제목]

### 고려한 선택지
- **A 방법**: [어떻게 해결하는지 문장으로 설명한다]
- **B 방법**: [어떻게 해결하는지 문장으로 설명한다]
- **C 방법**: [어떻게 해결하는지 문장으로 설명한다]

### ⭐ 대안 비교
| 선택지 | 장점 | 단점 |
|---------|------|------|
| A 방법 | ... | ... |
| B 방법 | ... | ... |
| **C 방법 (선택)** | ... | - |

### 선택한 방법과 이유
[왜 이 방법을 선택했는지 - 장단점 비교 기반으로 2-3줄]

### 핵심 아이디어
[선택한 방법의 구체적인 접근법 - 3-5줄]

### 단계별 접근 (선택)
1. 첫 번째로 한 것
2. 두 번째로 한 것
3. ...

### 구현 코드
\`\`\`typescript
// 실제 구현한 코드 예시 (변경된 파일에서 발췌)
// 주요 로직을 보여주는 부분
\`\`\`

## 📱 [다른 해결책] (필요시)
...
```

---

#### ✅ 결과

**목적**: 해결 후 달성한 것을 정리

**포함할 내용:**
- 각 플랫폼/케이스별 결과
- 성능/UX 개선 사항
- 추가 처리 사항

**예시:**
```markdown
## 🖥️ Desktop
- 한 번에 불러오고 → 날짜별로 필터링
- 월 전환 시에만 새로 fetch

## 📱 Mobile
- 듀얼 쿼리로 헤더/상세 분리
- 캐싱 덕분에 API 호출 최소화
```

---

#### 📚 배운점

**목적**: 기술적 인사이트와 실용적 팁 정리

**카테고리별 구조:**
```markdown
## 🔧 기술적 배움 (2-3개)
1. **[배움 제목]**
   - 세부 내용
   - 적용 팁

## 🧠 설계적 배움 (1-2개)
1. **[배움 제목]**
   - 세부 내용

## 💭 UX적 배움 (0-1개, 선택)
1. **[배움 제목]**
   - 세부 내용
```

---

#### 🤔 더 알아보기

**목적**: AI가 제안하는 추가 학습/개선 포인트

**포함할 내용:**
- **현재 방식의 대안**: 다른 접근법 비교 (SSR vs CSR 등)
- **더 깊이 공부하려면**: 관련 개념, 공식 문서, 추천 아티클
- **비슷한 상황에서 쓸 수 있는 대안**: 다른 라이브러리/패턴
- **성능/보안/유지보수 관점**: 개선할 수 있는 부분
- **결론 및 권장 사항**: 현재 선택의 타당성, 향후 고려사항

**예시:**
```markdown
### SSR vs CSR 비교
현재는 CSR로 구현했지만, SSR도 고려할 수 있다.

| 항목 | CSR (현재) | SSR (대안) |
|------|-----------|------------|
| 초기 로딩 | 클라이언트 fetch | 서버에서 미리 fetch |
| 복잡도 | 낮음 | 높음 |

### 결론
현재 구현도 충분하지만, 초기 로딩 UX 개선이 필요하면 SSR 고려.
```

---

#### 🔗 관련 링크

**목적**: 참고 자료 제공

**포함할 내용:**
- 공식 문서 링크
- 관련 아티클
- 사용한 라이브러리 문서

```markdown
- [React Query - Query Keys](https://tanstack.com/query/...)
- [Next.js - Data Fetching](https://nextjs.org/docs/...)
```

---

**작성 기준 요약:**
| 섹션 | 최소 분량 | 포함해야 할 것 |
|------|----------|---------------|
| 📋 배경 | 3-5줄 | 상황, 문제 인식, 목표 |
| 🔍 문제 | 각 문제당 5줄+ | 현상, 원인 분석 |
| 💡 해결 | 10줄+ | 고려한 선택지, **⭐ 대안 비교**, 선택 이유, 핵심 아이디어, 코드 |
| ✅ 결과 | 3-5줄 | 달성 사항, 개선점 |
| 📚 배운점 | 5줄+ | 기술/설계/UX 인사이트 |
| 🤔 더 알아보기 | 3-5줄 | 대안, 공식 문서, 개선점 |
| 🔗 관련 링크 | 2-4개 | 공식 문서, 참고 자료 |

---

**💡 작성 팁:**
1. **구체적으로 쓰기**: "문제가 있었다" ❌ → "isLoading만으로 제어해도 null이 순간 보였다" ✅
2. **코드 포함하기**: 핵심 로직은 반드시 코드 스니펫으로
3. **비교 테이블 활용**: 선택지가 여러 개면 테이블로 장단점 비교
4. **이모지 활용**: 섹션 구분과 가독성을 위해
5. **나중의 나를 위해**: 3개월 후에 봐도 이해할 수 있게 작성

---

**❌ 나쁜 예시:**
```markdown
## 🔍 문제
로그인 기능이 필요했다.

## 💡 해결
로그인 기능을 구현했다.

## 📚 배운점
React를 배웠다.
```

**✅ 좋은 예시:** (Desktop/Mobile Calendar 로딩 상태 제어)
```markdown
# 📋 배경

## 상황
ZeroGravity의 프로필 페이지에는 **감정 캘린더**가 있다.
- **Desktop**: 월별 캘린더 + 사이드 드로워
- **Mobile**: 주별 캘린더 + 하단 상세 영역
- **공통**: 드로워에 PlanetScene(3D 감정 행성)이 포함됨

## 문제 인식
날짜를 전환할 때 **로딩 상태 제어**가 필요했다.

## 목표
**데이터가 없을 때 깜빡임 없이 부드럽게 전환되는 UX**를 만드는 것.

---

# 🔍 문제

## ⚠️ 1. PlanetScene과 r3f 로딩의 결합

### 현상
PlanetScene 컴포넌트는 **r3f(React Three Fiber) 로딩**과 물려있었다.
서버 상태(API 데이터)와 연결하기가 어려웠다.

### 원인 분석: 순차적 의존성
\`\`\`
서버 데이터(emotionId) → r3f 렌더링(행성 형태 결정)
\`\`\`
`emotionId`가 있어야 어떤 행성을 그릴지 알 수 있다.
**서버 로딩이 끝나야 r3f 로딩이 시작될 수 있는 순차적 의존성**이 있어서
둘을 합친다는 게 구조적으로 어려웠다.

---

# 💡 해결

## 🖥️ Desktop: 기존 구조 유지하면서 해결

### 고려한 선택지
- **r3f에서 로딩 도려내기**: r3f 로딩을 서버 로딩과 분리해서 각각 독립적으로 제어한다
- **새로운 로딩 UI 추가**: 별도 로딩 오버레이를 만들어서 로딩 상태를 명시적으로 표시한다
- **날짜 데이터 선별 + 클릭 제어**: 월별 데이터를 한 번에 가져온 뒤 클라이언트에서 필터링한다

### ⭐ 대안 비교
| 선택지 | 장점 | 단점 |
|---------|------|------|
| r3f에서 로딩 도려내기 | 각각 독립적 제어 가능 | 리팩토링 부담 + 순차적 의존성으로 분리가 자연스럽지 않음 |
| 새로운 로딩 UI 추가 | 명확한 로딩 표시 | 레이어 복잡도 증가 |
| **날짜 데이터 선별 + 클릭 제어 (선택)** | 기존 구조 유지, 최소 변경 | - |

### 선택한 방법과 이유
r3f 로딩을 별도로 제어하지 않아도 되는 이유를 발견했다:
\`\`\`
1. 월 전환 시 새 데이터 fetch 시작
2. isLoading = true → 캘린더 셀 클릭 비활성화
3. 클릭이 안 되니까 드로워가 열리지 않음
4. 드로워가 안 열리니까 PlanetScene도 안 보임
5. 결론: r3f 로딩을 별도로 처리할 필요가 없다!
\`\`\`
**구조적 한계를 인정하고 우회하는 것도 좋은 선택**이었다.

### 핵심 아이디어
서버 로딩 중에는 어차피 드로워가 안 보이니까, 기존 r3f 구조를 유지하면서
로딩 상태만 캘린더 셀 클릭 비활성화로 제어.

---

## 📱 Mobile: 듀얼 쿼리 + 캐싱 전략

### 고려한 선택지
- **단일 쿼리 + 필터링**: Desktop처럼 주별 데이터를 한 번에 가져와서 필터링한다
- **듀얼 쿼리**: 헤더용 쿼리와 상세용 쿼리를 분리해서 각각 관리한다

### ⭐ 대안 비교
| 선택지 | 장점 | 단점 |
|---------|------|------|
| 단일 쿼리 + 필터링 | 쿼리 하나로 단순 | 주 전환 시 선택 날짜가 범위 밖이면 데이터 없음 |
| **듀얼 쿼리 (선택)** | 헤더/상세 데이터 분리 | 쿼리가 2개... 인 것 같지만 아님! |

### 선택한 방법과 이유
**핵심 인사이트: 쿼리 2개 = API 2번이 아니다!**
React Query의 Query Key 기반 캐싱 덕분에:
- 같은 주를 선택하면 → Query Key 동일 → **캐시 히트 → API 호출 0회**
- 다른 주를 선택하면 → 해당 주만 fetch → **API 호출 1회**

### 구현 코드
\`\`\`typescript
/** 현재 주 데이터 - 헤더 날짜 표시용 */
const { data: currentWeekRecords } = useGetEmotionRecordsQuery({
  startDateTime: format(currentWeekStart, \"yyyy-MM-dd'T'HH:mm:ss\"),
  endDateTime: format(currentWeekEnd, \"yyyy-MM-dd'T'HH:mm:ss\"),
});

/** 선택된 날짜의 주 데이터 - 상세 표시용 (같은 주면 캐시 히트!) */
const { data: selectedWeekRecords } = useGetEmotionRecordsQuery({
  startDateTime: format(selectedWeekStart, \"yyyy-MM-dd'T'HH:mm:ss\"),
  endDateTime: format(selectedWeekEnd, \"yyyy-MM-dd'T'HH:mm:ss\"),
});
\`\`\`

---

# ✅ 결과

## 🖥️ Desktop
- 기존 r3f 구조 유지하면서 로딩 문제 해결
- 월 전환 시에만 새로 fetch

## 📱 Mobile
- 듀얼 쿼리로 헤더/상세 역할 분리
- 캐싱 덕분에 실질적 API 호출은 최소화

---

# 📚 배운점

## 🔧 기술적 배움
1. **React Query의 Query Key 기반 캐싱이 강력하다**
   - 쿼리를 여러 개 선언해도 Query Key가 같으면 캐시 히트
   - "쿼리 2개 = API 2번"이 아님을 이해하는 게 중요

## 🧠 설계적 배움
1. **순차적 의존성이 있으면 로딩 통합이 어렵다**
   - `서버 데이터 → r3f 렌더링` 같은 의존 관계
   - 무리하게 통합하기보다 흐름을 자연스럽게 분리하는 게 나음

2. **구조적 한계를 인정하고 우회하는 것도 좋은 선택**
   - "드로워가 안 열리니까 어차피 안 보임" → 기존 구조 유지하면서 해결
   - 완벽한 구조보다 현실적인 해결책이 더 가치 있을 때가 있음

---

# 🤔 더 알아보기

### SSR로 초기 데이터를 가져왔어야 했나?
| 항목 | CSR (현재) | SSR (대안) |
|------|-----------|------------|
| 초기 로딩 | 클라이언트 fetch 후 표시 | 서버에서 미리 fetch |
| 구현 복잡도 | 낮음 | 높음 (hydration 고려) |

**결론**: 현재 구현도 충분. 초기 로딩 UX 개선 필요시 SSR 하이브리드 고려.

---

# 🔗 관련 링크
- [React Query - Query Keys](https://tanstack.com/query/...)
- [React Query - Caching](https://tanstack.com/query/...)
```

### Create Page

**Property Type Formats:**

| Type | Format | Example |
|------|--------|---------|
| title | string | `"제목": "로그인 기능 구현"` |
| date | expanded | `"date:날짜:start": "2025-01-01"`, `"date:날짜:is_datetime": 0` |
| select | string | `"타입": "feat"` |
| multi_select | **JSON array string** | `"기술 스택": "[\"React\", \"TypeScript\"]"` |
| rich_text | string (markdown OK) | `"프로젝트": "[repo](url)"` |

⚠️ **multi_select는 반드시 JSON 배열 문자열로 전달해야 함!**
- ❌ `"React, TypeScript"` (쉼표 구분 문자열)
- ✅ `"[\"React\", \"TypeScript\"]"` (JSON 배열 문자열)

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

### Step 6: Blog Extension Prompt

After TIL creation success, ask user:

**Ask user (AskUserQuestion):**
"블로그로 확장할까요?"

| Option | Description |
|--------|-------------|
| 예 | 지금 바로 블로그로 확장 |
| 나중에 | TIL만 기록하고 종료 |

**If "예":**
```
💡 블로그 확장을 위해 `/notion:blog` 를 실행하세요.
방금 작성한 TIL을 기반으로 더 자세한 블로그 글을 작성합니다.

TIL URL: [page URL]
```

**If "나중에":**
→ End with success message above

### On Failure:
```
✗ Failed to record TIL: <error message>
```

## Constraints

1. **Korean content**: TIL content in Korean
2. **Accurate tech stack**: Only include tech actually used in this work
3. **Deep analysis**: Read actual file content, don't just rely on diffs
4. **Git context reuse**: When from `/git:commit`, reuse analysis to save tokens
5. **DB naming**: Always use `[CLAUDE] TIL` for database name
6. **MCP required**: Cannot work without Notion MCP connection
7. **User-level config**: Store DB ID in `~/.claude/notion.json`, not project-level
