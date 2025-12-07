---
description: Create a GitHub Pull Request (light mode - git commands only, saves tokens)
---

# Create GitHub Pull Request - Light Mode

Analyze changes using git commands only and create a PR using `gh` CLI.
This is the light version that saves tokens by skipping file content reading and conversation context analysis.

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

## Step 3: Determine Target Branch

Get all remote branches (excluding current branch):

```bash
git branch -r | grep -v HEAD | sed 's/origin\///' | grep -v "^  $(git branch --show-current)$"
```

Present branches as numbered options to user:

```
Select target branch:
1. main
2. develop
3. feature/other-feature
...
N. Custom (enter branch name)
```

- Exclude current branch from the list
- Always add "Custom" as the last option
- Default: `main` or `master` (whichever exists)

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

## Step 5: Analysis of Changes (Light)

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

**Note:** This light mode does NOT read file contents or analyze conversation context to save tokens.

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
## Overview

[2-3 sentences summarizing the purpose and background of this PR based on commit and diff analysis]

## Key Changes

- [Specific change 1 based on actual code changes]
- [Specific change 2 based on actual code changes]
- [Specific change 3 based on actual code changes]

## Technical Details

- **Files Changed**: [number] files
- **Dependencies**: [New dependencies added, or "None"]
- **Breaking Changes**: [Description or "None"]

## Review Points

- [Area that needs careful review 1]
- [Area that needs careful review 2]

## Testing

- [ ] Build successful
- [ ] Type check passed
- [ ] Lint check passed
- [ ] Manual testing completed
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
3. **Git commands only**: Do not use Read tool to read file contents
4. **English only**: All PR content in English
5. **Use gh CLI**: Do not use GitHub API directly
6. **Max 50 commits**: If more than 50 commits, analyze only the most recent 50
