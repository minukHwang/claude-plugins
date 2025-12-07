---
description: Create a new git branch with type/description format and checkout
---

# Create Git Branch

Create a new branch following the naming convention and automatically checkout.

## Branch Naming Convention

**Personal project:**
```
<type>/<description>
```

**Team/Collaborative project:**
```
<accountName>/<type>/<description>
```

## Step 1: Ask for Branch Type (Cascading Selection)

Use cascading selection to show all branch types:

**Available types:** `feature` `bugfix` `hotfix` `release` `docs` `refactor` `test` `chore`

### Round 1 - Common Types

**Ask user (AskUserQuestion):**
"Select the branch type:"

| Option | Description |
|--------|-------------|
| feature | Adding a new feature or capability |
| bugfix | Fixing a bug or issue |
| hotfix | Urgent fix for production |
| More types | Show more options... |

### Round 2 - If "More types" selected

**Ask user (AskUserQuestion):**
"Select the branch type:"

| Option | Description |
|--------|-------------|
| release | Release preparation |
| docs | Documentation changes |
| refactor | Code restructuring |
| More types | Show remaining options... |

### Round 3 - If "More types" selected again

**Ask user (AskUserQuestion):**
"Select the branch type:"

| Option | Description |
|--------|-------------|
| test | Testing related changes |
| chore | Routine maintenance tasks |

User can also enter custom type via "Other" option (kebab-case).

## Step 2: Ask for Description

Ask the user:
"Enter a brief description for the branch (use kebab-case, e.g., emotion-calendar):"

### Description Rules
- Use **kebab-case** (lowercase with hyphens)
- Keep it **short and descriptive** (2-5 words)
- No special characters except hyphens
- Examples: `emotion-calendar`, `auth-redirect-fix`, `readme-update`

## Step 3: Ask for Project Type

Ask the user:
"Is this a collaborative/team project?"

**Options:**
- **Yes**: Include GitHub account name in branch (e.g., `minukHwang/feature/emotion-calendar`)
- **No**: Use simple format (e.g., `feature/emotion-calendar`)

### Get GitHub Account Name (if collaborative)

If collaborative project, get the GitHub username:

```bash
gh api user -q .login
```

If `gh` is not available or fails, ask the user:
"Enter your GitHub username:"

## Step 4: Generate Branch Name

Construct the branch name:

**Personal project:**
```
{type}/{description}
```

**Collaborative project:**
```
{accountName}/{type}/{description}
```

### Examples

| Project Type | Type | Description | Branch Name |
|--------------|------|-------------|-------------|
| Personal | feature | emotion-calendar | `feature/emotion-calendar` |
| Personal | bugfix | auth-redirect | `bugfix/auth-redirect` |
| Collaborative | feature | user-profile | `minukHwang/feature/user-profile` |
| Collaborative | hotfix | login-crash | `minukHwang/hotfix/login-crash` |

## Step 5: Create and Checkout Branch

Create the branch and switch to it:

```bash
git checkout -b <branch-name>
```

## Output Format

### On Success:
```
✓ Branch created and checked out: <branch-name>

You can now start working on this branch.
When ready, use `/git:commit` to commit your changes.
```

### On Failure:
```
✗ Failed to create branch: <error message>
```

### Common Errors:
- **Branch already exists**: "Branch '<name>' already exists. Use a different name or checkout the existing branch with `git checkout <name>`"
- **Invalid branch name**: "Invalid branch name. Please use only lowercase letters, numbers, and hyphens."

## Constraints

1. **Use full type names**: `feature` not `feat`, `bugfix` not `fix`
2. **kebab-case only**: No spaces, underscores, or uppercase
3. **English only**: All branch names in English
4. **Auto-checkout**: Always switch to the new branch after creation
