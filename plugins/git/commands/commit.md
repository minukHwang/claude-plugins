---
description: Generate conventional commit message from staged changes and commit
---

# Git Commit with Conventional Commits

Analyze staged changes and generate a conventional commit message in English.

## Step 1: Check Staged Changes

First, verify there are staged changes:

```bash
git diff --staged --quiet
```

If no staged changes exist, output this message and stop:
"No staged changes found. Please use `git add <file>` to stage your changes first."

## Step 2: Analyze Changes

Analyze the staged changes:

```bash
git diff --staged --stat
git diff --staged
```

Analyze:
- Number of changed files
- Nature of changes (new feature, bug fix, refactoring, etc.)
- Affected modules/components

## Step 3: Generate Commit Message

### Commit Message Format

**Format 1: Single line (for simple changes with 1-2 files)**
```
<type>: <description>
```

**Format 2: With bullet points (for complex changes with 3+ files or multiple changes)**
```
<type>: <description>

- Detail 1
- Detail 2
- Detail 3
```

### Determine Commit Type

| Type | Description |
|------|-------------|
| feat | Adding a new feature or capability |
| fix | Resolving a bug or issue |
| docs | Updating documentation or comments |
| style | Formatting code without changing functionality |
| refactor | Restructuring existing code |
| perf | Optimizing performance |
| test | Adding or modifying tests |
| build | Modifying build tools or dependencies |
| ci | Updating CI/CD configuration |
| chore | Routine maintenance tasks |
| revert | Undoing a previous commit |
| init | Initial project setup |

### Rules

- **50 characters max** for the main description
- Use **imperative mood** (e.g., "Add", "Fix", "Update", not "Added", "Fixed")
- **No period** at the end
- **English only**
- **DO NOT include emoji** - gitmoji hook will add it automatically
- Choose format based on change complexity:
  - Simple (1-2 files, single purpose) → Single line
  - Complex (3+ files or multiple changes) → With bullet points

### Examples

**Single line:**
```
feat: Add emotion calendar view with monthly navigation
fix: Resolve NextAuth session refresh on token expiration
docs: Update installation guide in README
```

**With bullet points:**
```
feat: Add user authentication system

- Implement Google OAuth provider
- Add session management with JWT
- Create login and logout pages
```

## Step 4: Execute Commit

Commit with the generated message:

```bash
# Single line format
git commit -m "<type>: <description>"

# With bullet points format
git commit -m "<type>: <description>" -m "- Detail 1
- Detail 2
- Detail 3"
```

## Output Format

### On Success:
```
✓ Commit created: <commit hash>
  <commit message>
```

### On Failure:
```
✗ Commit failed: <error message>
```

## Constraints

1. **NEVER include**: "Generated with Claude Code"
2. **NEVER include**: "Co-Authored-By"
3. **DO NOT run git add**: Only commit already staged changes
4. **English only**: All text must be in English
5. **NO emoji in message**: gitmoji hook handles this automatically
