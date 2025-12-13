---
description: Create a GitHub Pull Request with detailed analysis of changes
---

# Create GitHub Pull Request

Analyze all changes in the current branch and create a comprehensive PR using `gh` CLI.

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

#### 1. **Feature Category 1**

- **Specific Feature**: Description of the feature
- **Technical Detail**: Implementation details
- **User Impact**: How this affects the user experience

#### 2. **Feature Category 2**

- **Specific Feature**: Description of the feature
- **Technical Detail**: Implementation details
- **User Impact**: How this affects the user experience

#### 3. **Technical Improvements**

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

### 🔗 **Related Issues**

Closes #(issue number)
Related to #(issue number)
```

### Body Writing Guidelines

- **Overview**: Focus on business value or problem solved, not implementation details
- **Key Changes**: Be specific - mention actual file names, components, or functions
- **Technical Details**: Include concrete numbers and facts
- **Review Points**: Highlight risky areas or complex logic that needs attention

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
