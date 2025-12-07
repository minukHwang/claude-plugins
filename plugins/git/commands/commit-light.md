---
description: Generate conventional commit message (light mode - git diff only, saves tokens)
---

# Git Commit - Light Mode

Analyze staged changes using git diff only and generate a conventional commit message.
This is the light version that saves tokens by skipping file content reading and conversation context analysis.

## Step 0: Detect Project Commit Rules

Automatically search for commitlint configuration (no user prompt):

```bash
# Search for commitlint config files
ls commitlint.config.{js,cjs,mjs,ts} .commitlintrc.{json,yaml,yml,js,cjs} .commitlintrc 2>/dev/null
```

Also check `package.json` for a `commitlint` field.

### If config found:
- Read the configuration file
- Extract rules: allowed types, character limits, emoji requirements
- **MERGE** with plugin defaults:
  - Use THEIR: types, character limits (must pass validation)
  - Use OUR: emoji prefix format (`✨ type: description`)

### If no config found:
- Use plugin defaults
- Include emoji prefix
- 100 characters max
- Standard commit types

## Step 1: Check and Stage Changes

First, check the staging status:

```bash
git status --short
```

Parse the output to determine:
- Staged files (lines starting with `M `, `A `, `D `, `R `)
- Unstaged files (lines starting with ` M`, ` D`, `??`)

### Case A: All changes are staged (no unstaged files)
Proceed to Step 2 without asking.

### Case B: Some files staged, some unstaged (partial staging)
Show what's currently staged and ask for confirmation:

```
📋 Currently staged:
  M src/components/Button.tsx
  A src/utils/helpers.ts

⚠️ Not staged:
  M src/App.tsx
  ?? src/new-file.ts
```

**Ask user:** "Proceed with only the staged files above?"

| Option | Description |
|--------|-------------|
| 1 | Yes, commit only staged files |
| 2 | Add all remaining changes too |
| 3 | Select specific files to add |

#### If Option 1: Proceed to Step 2
#### If Option 2: Run `git add .` then proceed
#### If Option 3: Let user specify files, run `git add`, then proceed

### Case C: Nothing staged (all files unstaged)
Show unstaged files and ask user to add:

**Ask user:** "No staged changes. What would you like to add?"

| Option | Description |
|--------|-------------|
| 1 | Add all changes (`git add .`) |
| 2 | Enter file path or pattern (e.g., `src/components/*.tsx`) |
| 3 | Describe what to add (e.g., "button related files") |

#### If Option 1 selected:
```bash
git add .
```

#### If Option 2 selected:
User enters file path/pattern, then:
```bash
git add <user-input>
```

#### If Option 3 selected:
User describes files contextually (e.g., "calendar related", "auth changes").
- Analyze `git status` output
- Match files based on description
- Show matched files and confirm with user
- Run `git add` for confirmed files

After adding, show what was staged:
```bash
git diff --staged --stat
```

If still no staged changes after add attempt, show error and stop.

## Step 2: Analyze Changes (Light)

Analyze the staged changes using git commands only:

```bash
git diff --staged --stat
git diff --staged
```

Analyze:
- Number of changed files
- Nature of changes (new feature, bug fix, refactoring, etc.)
- Affected modules/components

**Note:** This light mode does NOT read file contents or analyze conversation context to save tokens.

## Step 3: Generate Commit Message

### Commit Message Format

**Format 1: Single line (for simple changes with 1-2 files)**
```
<emoji> <type>: <description>
```

**Format 2: With bullet points (for complex changes with 3+ files or multiple changes)**
```
<emoji> <type>: <description>

- Detail 1
- Detail 2
- Detail 3
```

### Commit Type and Emoji Mapping

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

### Rules

- **100 characters max** for the main description (or follow commitlint limit if configured)
- Use **imperative mood** (e.g., "Add", "Fix", "Update", not "Added", "Fixed")
- **No period** at the end
- **English only**
- **Always include emoji** at the start of the message
- Choose format based on change complexity:
  - Simple (1-2 files, single purpose) → Single line
  - Complex (3+ files or multiple changes) → With bullet points

### Examples

**Single line:**
```
✨ feat: Add emotion calendar view with monthly navigation
🐛 fix: Resolve NextAuth session refresh on token expiration
📄 docs: Update installation guide in README
```

**With bullet points:**
```
✨ feat: Add user authentication system

- Implement Google OAuth provider
- Add session management with JWT
- Create login and logout pages
```

## Step 4: Execute Commit

Commit with the generated message:

```bash
# Single line format
git commit -m "<emoji> <type>: <description>"

# With bullet points format
git commit -m "<emoji> <type>: <description>" -m "- Detail 1
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
3. **English only**: All text must be in English
4. **Always include emoji**: Add the appropriate gitmoji at the start of the message
5. **Confirm before add**: Always show what will be staged before running git add
