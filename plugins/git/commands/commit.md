---
description: Generate conventional commit message from staged changes and commit
---

# Git Commit with Conventional Commits

Analyze staged changes and generate a conventional commit message in English.

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
  - Priority: pass commitlint validation > exact match our format

### If no config found:
- Use plugin defaults (see below)
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

**Ask user (AskUserQuestion):** "Proceed with only the staged files above?"

| Option | Description |
|--------|-------------|
| Yes, staged only | Commit only staged files |
| Add all | Add all remaining changes too |
| Select files | Let me specify files to add |

#### If "Yes, staged only": Proceed to Step 2
#### If "Add all": Run `git add .` then proceed
#### If "Select files": Let user specify files, run `git add`, then proceed

### Case C: Nothing staged (all files unstaged)
Show unstaged files and ask user to add:

**Ask user (AskUserQuestion):** "No staged changes. What would you like to add?"

| Option | Description |
|--------|-------------|
| Add all | Add all changes (`git add .`) |
| Enter path | Enter file path or pattern |
| Describe | Describe what to add contextually |

#### If "Add all" selected:
```bash
git add .
```

#### If "Enter path" selected:
User enters file path/pattern, then:
```bash
git add <user-input>
```

#### If "Describe" selected:
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

## Step 1.5: React File Detection (Optional)

Check if staged files include React/TypeScript files:

```bash
git diff --staged --name-only | grep -E '\.(tsx)$'
```

### If `.tsx` files are found:

**Ask user (AskUserQuestion):**
"📝 React files detected. Would you like to organize comments first?"

| Option | Description |
|--------|-------------|
| Yes | Run `/react:comment` then continue |
| No | Proceed with commit |

#### If "Yes":
1. Run `/react:comment` with staged `.tsx` files
2. Re-stage modified files: `git add <modified-files>`
3. Continue to Step 2

#### If "No":
Continue to Step 1.6

### If no `.tsx` files:
Skip to Step 1.6

## Step 1.6: Initial Commit Detection

Check if this is the first commit in the repository:

```bash
git rev-list --count HEAD 2>/dev/null || echo "0"
```

### If count is 0 (no commits yet):

This is an initial commit. Auto-configure:

```
🎉 Initial commit detected!
   Suggested type: init
   Suggested message: Initialize {repo-name}
```

Get repository name:
```bash
basename $(pwd)
```

**Pre-configure for Step 3:**
- Type: `init`
- Emoji: 🎉

**Continue to Step 2** for analysis:
- Analyze all staged files (structure, components, features)
- Check conversation context (what was the user building?)
- Generate appropriate description

Description examples based on analysis:
- Simple boilerplate: `Initialize {repo-name}`
- With features: `Initialize project with Next.js, TypeScript, and authentication`
- From conversation: `Initialize emotion tracking app with calendar and 3D visualization`

### If count > 0:
Continue to Step 2 (normal commit flow)

## Step 2: Deep Analysis of Changes

### 2.1 Diff Analysis
```bash
git diff --staged --stat
git diff --staged
```

### 2.2 File Content Analysis
For key changed files, read the actual content to understand context:
- Use Read tool to examine important changed files
- Understand the purpose and impact of changes
- Identify patterns, new components, or modified logic

### 2.3 Conversation Context Analysis
If there's relevant conversation history in this session:
- What task was the user working on?
- What problem were they solving?
- Any specific feature names or terminology mentioned?
- User's intent behind the changes

### 2.4 Combined Analysis
Synthesize all information:
- Number of changed files
- Nature of changes (new feature, bug fix, refactoring, etc.)
- Affected modules/components
- User's original intent from conversation

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

## Step 5: TIL Suggestion (Optional)

After successful commit:

**Ask user (AskUserQuestion):**
"📝 Record TIL to Notion?"

| Option | Description |
|--------|-------------|
| Yes | Record TIL to Notion |
| No | Skip |

### If "Yes":
Run `/notion:til` command.
- Reuse the analysis context from this commit (saves tokens)
- The TIL will be based on the commit just made

### If "No":
Proceed to output

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
