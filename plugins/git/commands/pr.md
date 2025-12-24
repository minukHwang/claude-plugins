---
description: Create a GitHub Pull Request with detailed analysis of changes
---

# Create GitHub Pull Request

Analyze all changes in the current branch and create a comprehensive PR using `gh` CLI.

## Step 0: Load Workflow Configuration (Optional)

```bash
cat .claude/workflow.json 2>/dev/null
```

If exists, extract:
- `jira.enabled` - Whether Jira integration is active
- `jira.cloudId` - Atlassian cloud ID for Jira URLs
- `jira.siteUrl` - Atlassian site URL for Jira links
- `notion.enabled` - Whether Notion integration is active

If `notion.enabled`, also load:
```bash
cat ~/.claude/notion.json 2>/dev/null
```

Extract `todo.id` for Notion TODO updates.

## Step 1: Verify Git Repository

Check if this is a git repository:

```bash
git rev-parse --git-dir
```

If not a git repository: "This is not a git repository." and stop.

## Step 2: Get Current Branch

```bash
git branch --show-current
```

Store as `SOURCE_BRANCH`.

If detached HEAD: "No branch checked out. Please checkout a branch first." and stop.

If on `main` or `master`: "You are on the main branch. Please checkout a feature branch first." and stop.

## Step 2.5: Extract Jira Issue Key (if jira.enabled)

If workflow.json has `jira.enabled: true`:

Extract issue key from current branch:
```bash
echo "$SOURCE_BRANCH" | grep -oE '[A-Z]+-[0-9]+'
```

If issue key found (e.g., `CP-1`):
- Store for PR body generation
- Will create Jira link: `https://{cloudId}/browse/{issueKey}`

Get issue details for PR context:
Call `mcp__atlassian__getJiraIssue`:
```
cloudId: {jira.cloudId}
issueIdOrKey: {issueKey}
fields: ["summary", "status", "issuetype"]
```

## Step 3: Determine Target Branch (Cascading Selection)

Get all remote branches (excluding current branch):

```bash
git branch -r | grep -v HEAD | sed 's/origin\///' | grep -v "^  $(git branch --show-current)$"
```

### Round 1 - Common Target Branches

**Ask user (AskUserQuestion):**
"Select target branch:"

| Option | Description |
|--------|-------------|
| main | Default branch (or master if main doesn't exist) |
| develop | Development branch (if exists) |
| Other | Show more branches... |

### Round 2+ - If "Other" selected
Show remaining branches in groups of 3:

**Ask user (AskUserQuestion):**
"Select target branch:"

| Option | Description |
|--------|-------------|
| 1 | {branch-1} |
| 2 | {branch-2} |
| 3 | {branch-3} |
| 4 | Other - Show more... |

### Final Round
When remaining branches are 3 or fewer:

**Ask user (AskUserQuestion):**
"Select target branch:"

| Option | Description |
|--------|-------------|
| 1 | {remaining-branch-1} |
| 2 | {remaining-branch-2} |
| 3 | {remaining-branch-3} (if exists) |

User can also enter custom branch name via "Other" option.

### Verification
Verify selected target branch exists:
```bash
git rev-parse --verify origin/<TARGET_BRANCH>
```

If not exists: "Target branch does not exist." and stop.

## Step 4: Push Current Branch

Ensure the branch is pushed to remote:

```bash
git push -u origin HEAD
```

If push fails, show the error and stop.

## Step 5: Deep Analysis of Changes

### 5.1 Commit Analysis
Get all commit messages since branching:

```bash
git log origin/<TARGET_BRANCH>..HEAD --pretty=format:"%s%n%b%n---" --reverse
```

### 5.2 Diff Analysis
Get the complete diff:

```bash
git diff origin/<TARGET_BRANCH>..HEAD
```

### 5.3 Changed Files Analysis
Get list of changed files:

```bash
git diff origin/<TARGET_BRANCH>..HEAD --name-only
```

### 5.4 File Content Analysis
For each changed file, read the actual content to understand the context:
- Use Read tool to examine key changed files
- Focus on understanding the purpose and impact of changes
- Identify patterns, new components, or modified logic

### 5.5 Conversation Context Analysis
If there's relevant conversation history in this session:
- What task was the user working on?
- What problem were they solving?
- Any specific feature names or terminology mentioned?
- User's original intent and requirements
- Discussions about implementation approach

### 5.6 Combined Analysis
Synthesize all information from:
- Commit messages (intent)
- Git diff (code changes)
- File content (context)
- Conversation history (user's goals)

### 5.7 Version Update Check (claude-plugins only)

**Only for claude-plugins repository.**

Check if plugin files changed (from Step 5.3 changed files):
- `plugins/*/commands/*.md` changed?
- `plugins/*/skills/*.md` changed?

If detected:

**Ask user (AskUserQuestion):**
"🏷️ Plugin changes detected. Version update needed?"

| Option | Description |
|--------|-------------|
| PATCH (X.Y.Z+1) | Bug fix, docs improvement |
| MINOR (X.Y+1.0) | New feature added |
| Skip | Keep current version |

### If PATCH or MINOR selected:
1. Read current version from `package.json`
2. Update version in these files:
   - `package.json`
   - `.claude-plugin/marketplace.json` (metadata.version + plugins[affected].version)
   - `plugins/{affected}/.claude-plugin/plugin.json`
3. Create version commit:
   `:bookmark: release: v{new_version}`
4. Push and continue to Step 6 (version commit included in PR)

### 5.8 Multi-Issue Warning (if jira.enabled)

Extract all issue keys from commit messages:
```bash
git log origin/<TARGET_BRANCH>..HEAD --pretty=format:"%s %b" | grep -oE '[A-Z]+-[0-9]+' | sort -u
```

Compare with branch issue key:
- Branch key: `{issueKey}` (from Step 2.5)
- Commit keys: All unique keys from commits

If commit keys contain keys OTHER than branch key:

```
⚠️ Warning: Multiple issue keys detected!

Branch: {branch_issue_key}
Commits reference: {all_issue_keys}

Best practice: 1 PR = 1 Issue
```

**Ask user (AskUserQuestion):**
"Multiple Jira issues found in commits. Continue?"

| Option | Description |
|--------|-------------|
| Continue | Create PR anyway (not recommended) |
| Cancel | Abort to fix commits |

If "Cancel" selected, stop with message:
```
PR creation cancelled.

To fix, consider:
- Cherry-pick relevant commits to separate branches
- Create separate PRs for each issue
- Use /jira:start to work on one issue at a time
```

## Step 6: Generate PR Title

Format: `<emoji> <type>: <description>`

### Commit Type and Emoji Mapping (Same as commits)

| Type | Emoji | Description |
|------|-------|-------------|
| feat | ✨ | Adding a new feature or capability |
| fix | 🐛 | Resolving a bug or issue |
| docs | 📄 | Updating documentation or comments |
| style | 🎨 | Formatting code without changing functionality |
| refactor | 📦 | Restructuring existing code |
| perf | 🚀 | Optimizing performance |
| test | 🚨 | Adding or modifying tests |
| build | 🔨 | Modifying build tools or dependencies |
| ci | 🔧 | Updating CI/CD configuration |
| chore | 📝 | Routine maintenance tasks |
| revert | 🗑 | Undoing a previous commit |
| init | 🎉 | Initial project setup |
| release | 🔖 | Version release or tagging |

### Type Selection
Choose the **dominant type** from all commits in this PR.

### Title Rules
- **100 characters max**
- Use **gitmoji + type** format (same as commits)
- **Single dominant type**
- **English, concise**

### Examples
- `✨ feat: Add emotion calendar with monthly navigation`
- `🐛 fix: Resolve NextAuth session refresh on token expiration`
- `📄 docs: Update README installation guide`

## Step 7: Generate PR Body

Use this template:

```markdown
## 📝 **PR Description**

### 🎯 **Overview**

Brief description of what this PR implements or fixes.

### ✨ **Key Features**

<!-- Add as many feature categories as needed (1, 2, 3, 4...) -->

#### 1. **[Feature Category Name]**

- **[Specific Feature]**: Description of the feature
- **[Technical Detail]**: Implementation details
- **[User Impact]**: How this affects the user experience

#### 2. **[Feature Category Name]**

- **[Specific Feature]**: Description
- **[Technical Detail]**: Implementation details

<!-- Add more categories as needed -->

#### N. **Technical Improvements** (if applicable)

- **Performance**: Any performance improvements
- **Code Quality**: Code refactoring or improvements
- **Developer Experience**: DX improvements

### 🔧 **Technical Details**

- **Dependencies**: Any new dependencies added
- **Breaking Changes**: Any breaking changes (if applicable)
- **Migration Guide**: If needed, migration steps for breaking changes

### 🧪 **Testing**

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed
- [ ] Cross-browser testing (Chrome, Safari, Firefox)
- [ ] Responsive design tested (mobile/desktop)
- [ ] Build successful (`pnpm build:all`)
- [ ] Type check passed (`pnpm type-check:all`)
- [ ] Lint check passed (`pnpm lint:all`)

### 📸 **Screenshots/Videos**

<!-- Add screenshots or videos if UI changes are included -->

### 📋 **Checklist**

- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Documentation updated (if needed)
- [ ] No console errors or warnings
- [ ] Responsive design tested (if applicable)
- [ ] Accessibility checked (if UI changes)

### 🎫 **Jira Issue** (if jira.enabled)

[{ISSUE-KEY}](https://{cloudId}/browse/{ISSUE-KEY}) - {issue summary}

### 🔗 **Related Issues**

Closes #(issue number)
Related to #(issue number)
```

**Note:** The Jira Issue section should only be included if:
- workflow.json has `jira.enabled: true`
- Issue key was extracted from branch name
- Replace `{ISSUE-KEY}`, `{cloudId}`, and `{issue summary}` with actual values

### Body Writing Guidelines

- **Overview**: Focus on business value or problem solved, not implementation details
- **Key Features**: Be specific - mention actual file names, components, or functions
- **Technical Details**: Include concrete numbers and facts
- **Remove empty sections**: If no screenshots, related issues, or other optional content exists, remove the entire section (don't leave empty or "None" sections)

## Step 7.5: File Reference Prompt (Optional)

After PR body generated, before creating PR:

**Ask user (AskUserQuestion):**
"📁 Reference other files?"

| Option | Description |
|--------|-------------|
| Yes | Enter file path |
| No | Skip |

### If "Yes":

User enters file path → Read file with Read tool → Use content to enhance PR description (especially Technical Details section).

Example paths:
- `docs/plans/PLAN_feature.md` - Plan file
- `DECISIONS.md` - Decision log
- Any relevant documentation file

## Step 8: Create PR

Execute:

```bash
gh pr create --title "<title>" --body "<body>" --base <TARGET_BRANCH>
```

Use HEREDOC for the body to preserve formatting:

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Overview
...

## Key Changes
...
EOF
)" --base <TARGET_BRANCH>
```

## Step 8.5: Update Notion TODO with PR (if jira.enabled and notion.enabled)

After successful PR creation, if Jira integration is enabled:

1. Extract PR number from `gh pr create` output or URL

2. Load Notion config:
   ```bash
   cat ~/.claude/notion.json 2>/dev/null
   ```

3. Search for TODO item by Jira issue key:
   Call `mcp__notion__notion-search`:
   ```
   query: "{issueKey}"
   ```

4. If found, update PR field (rich text format like TIL):
   Call `mcp__notion__notion-update-page`:
   ```
   data: {
     "page_id": "{found_page_id}",
     "command": "update_properties",
     "properties": {
       "PR": "[#{pr_number}]({pr_url})"
     }
   }
   ```

**Note:** PR format follows TIL convention: `[#{number}]({url})`
Example: `[#42](https://github.com/user/repo/pull/42)`

## Step 9: README Update Suggestion (Optional)

After successful PR creation:

**Ask user (AskUserQuestion):**
"📄 Update README?"

| Option | Description |
|--------|-------------|
| Yes | Update README |
| No | Skip |

### If "Yes":
Run `/readme:update` command.
- Reuse the analysis context from this PR (saves tokens)
- Auto-select "PR changes" mode
- The update will reflect all changes in this PR

### If No:
Proceed to output

## Output Format

### On Success:
```
✓ Branch pushed to origin
✓ Pull Request created successfully!

Title: <PR title>
URL: <PR URL>

{if notion updated}
✓ Notion TODO updated
  └── PR: [#{pr_number}]({pr_url})
{/if}

The PR is ready for review.
```

### On Failure:
```
✗ Failed to create PR: <error message>
```

## Constraints

1. **NEVER include**: "Generated with Claude Code"
2. **NEVER include**: "Co-Authored-By"
3. **Full analysis**: Always analyze commits + diff + file content
4. **English only**: All PR content in English
5. **Use gh CLI**: Do not use GitHub API directly
6. **Max 50 commits**: If more than 50 commits, analyze only the most recent 50
