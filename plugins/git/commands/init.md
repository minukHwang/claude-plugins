---
description: Initialize git repository with hooks, .gitignore, and initial commit
---

# Initialize Git Project

Complete git project initialization: repository setup, .gitignore generation, commit tooling (husky/commitlint), and initial commit.

---

## Step 0: Git Repository Setup

### Check git initialization:
```bash
ls -d .git 2>/dev/null
```

### If .git exists:
Check remote configuration:
```bash
git remote -v
```

#### If remote exists:
```
✓ Git initialized
  Remote: origin → {url}
```
→ Continue to Step 1

#### If no remote:
**Ask user (AskUserQuestion):** "Connect to remote repository?"

| Option | Description |
|--------|-------------|
| Existing URL | Connect to existing GitHub repository |
| Create new | Create new repo on GitHub (gh CLI) |
| Skip | Continue without remote |

### If .git does NOT exist:
```bash
git init
```

**Ask user (AskUserQuestion):** "Connect to remote repository?"

| Option | Description |
|--------|-------------|
| Existing URL | Connect to existing GitHub repository |
| Create new | Create new repo on GitHub (gh CLI) |
| Skip | Continue without remote |

### Handle remote options:

#### Existing URL:
Get URL input from user, then:
```bash
git remote add origin <url>
```

#### Create new:
Check gh CLI:
```bash
gh --version 2>/dev/null
```

If not installed:
```
⚠️ GitHub CLI not installed.
Install: https://cli.github.com/
Falling back to "Skip" option.
```

If installed:

**Ask user (AskUserQuestion):** "Repository visibility?"

| Option | Description |
|--------|-------------|
| Public | Anyone can see this repository |
| Private | Only you can see this repository |

Get repository name (default: current folder name):
```bash
basename $(pwd)
```

Create repository:
```bash
# Public
gh repo create <repo-name> --public --source=. --remote=origin

# Private
gh repo create <repo-name> --private --source=. --remote=origin
```

#### Skip:
```
⏭️ Skipping remote setup. You can add later:
   git remote add origin <url>
```

---

## Step 1: Generate .gitignore

### Check existing .gitignore:
```bash
ls .gitignore 2>/dev/null
```

### If .gitignore exists:
**Ask user (AskUserQuestion):** "How to handle existing .gitignore?"

| Option | Description |
|--------|-------------|
| Merge | Add missing entries only |
| Replace | Replace with template |
| Skip | Keep existing |

### If .gitignore does NOT exist (or Replace selected):
Detect project type:
```bash
# Priority order
ls package.json 2>/dev/null && echo "nodejs"
ls requirements.txt pyproject.toml setup.py 2>/dev/null && echo "python"
ls go.mod 2>/dev/null && echo "go"
ls Cargo.toml 2>/dev/null && echo "rust"
ls Gemfile 2>/dev/null && echo "ruby"
ls *.xcodeproj 2>/dev/null && echo "ios"
ls build.gradle 2>/dev/null && echo "android"
```

### .gitignore Templates:

#### Node.js:
```gitignore
# Dependencies
node_modules/

# Build outputs
dist/
build/
.next/
out/

# Environment
.env
.env.local
.env.*.local

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Cache
.turbo/
.cache/
.eslintcache

# Test coverage
coverage/
```

#### Python:
```gitignore
# Byte-compiled
__pycache__/
*.py[cod]
*$py.class

# Virtual environments
venv/
.venv/
env/
.env/

# Environment variables
.env

# IDE
.idea/
.vscode/
*.swp

# OS
.DS_Store

# Distribution
dist/
build/
*.egg-info/

# Testing
.pytest_cache/
.coverage
htmlcov/

# Jupyter
.ipynb_checkpoints/
```

#### Go:
```gitignore
# Binaries
bin/
*.exe
*.exe~
*.dll
*.so
*.dylib

# Test binary
*.test

# Output
*.out

# Dependency directories
vendor/

# IDE
.idea/
.vscode/

# OS
.DS_Store

# Environment
.env
```

#### Rust:
```gitignore
# Build
/target/

# IDE
.idea/
.vscode/

# OS
.DS_Store

# Environment
.env
```

#### Ruby:
```gitignore
# Dependencies
vendor/bundle/
.bundle/

# Gems
*.gem

# Environment
.env

# IDE
.idea/
.vscode/

# OS
.DS_Store

# Logs
log/
*.log
```

#### iOS/macOS:
```gitignore
# Xcode
build/
DerivedData/
*.xcuserstate
*.xcworkspace/xcuserdata/

# CocoaPods
Pods/

# Carthage
Carthage/Build/

# SPM
.swiftpm/
.build/

# IDE
.idea/

# OS
.DS_Store

# Environment
.env
```

#### Android/Java:
```gitignore
# Build
build/
.gradle/

# Local configuration
local.properties

# IDE
.idea/
*.iml

# OS
.DS_Store

# Environment
.env
```

#### Generic (fallback):
```gitignore
# Environment
.env
.env.local

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
```

Show generated entries:
```
📄 .gitignore created ({type} template)
   Added {count} entries
```

---

## Step 2: Check Project Type

```bash
ls package.json 2>/dev/null
```

### If Node.js project:
→ Continue to Step 3 (Husky setup)

### If NOT Node.js:
Detect project type:
```bash
ls requirements.txt pyproject.toml setup.py 2>/dev/null && echo "Python"
ls go.mod 2>/dev/null && echo "Go"
ls Cargo.toml 2>/dev/null && echo "Rust"
ls Gemfile 2>/dev/null && echo "Ruby"
```

Show message:
```
📦 Detected: {Language} project

Git hooks options:
```

**Ask user (AskUserQuestion):** "How to set up git hooks?"

| Option | Description |
|--------|-------------|
| pre-commit | Install pre-commit framework (Python) |
| Manual | Create hooks manually in .git/hooks |
| Skip | Skip hooks, continue to initial commit |

#### pre-commit selected:
```bash
pip install pre-commit

# Create .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
EOF

pre-commit install
```

→ Skip to Step 11 (Initial Commit)

#### Manual selected:
```
📁 Manual hook setup:
   1. Create scripts in .git/hooks/
   2. Make executable: chmod +x .git/hooks/<hook-name>

   Common hooks:
   - pre-commit: Run before commit
   - commit-msg: Validate commit message
   - pre-push: Run before push
```

→ Skip to Step 11 (Initial Commit)

#### Skip selected:
→ Skip to Step 11 (Initial Commit)

---

## Step 3: Check Existing Husky Setup

```bash
ls .husky 2>/dev/null
ls commitlint.config.* .commitlintrc* 2>/dev/null
```

### If already set up:
Show current configuration:
```
⚠️ Existing setup detected:
  - .husky/ directory: ✓/✗
  - commitlint config: ✓/✗
  - prepare-commit-msg: ✓/✗
  - commit-msg: ✓/✗
  - pre-commit: ✓/✗
```

**Ask user (AskUserQuestion):** "How to proceed with existing setup?"

| Option | Description |
|--------|-------------|
| Skip | Keep existing configuration |
| Merge | Add missing parts only |
| Overwrite | Replace all configuration |

#### Skip: Exit husky setup → Go to Step 11
#### Merge: Only add missing hooks/config
#### Overwrite: Replace everything

---

## Step 4: Check Package Manager

```bash
ls pnpm-lock.yaml 2>/dev/null && echo "pnpm"
ls yarn.lock 2>/dev/null && echo "yarn"
ls package-lock.json 2>/dev/null && echo "npm"
```

If no lock file found:

**Ask user (AskUserQuestion):** "Which package manager?"

| Option | Description |
|--------|-------------|
| pnpm | Use pnpm |
| npm | Use npm |
| yarn | Use yarn |

---

## Step 5: Install Dependencies

### pnpm:
```bash
pnpm add -D husky @commitlint/cli @commitlint/config-conventional commitlint-config-gitmoji
```

### npm:
```bash
npm install -D husky @commitlint/cli @commitlint/config-conventional commitlint-config-gitmoji
```

### yarn:
```bash
yarn add -D husky @commitlint/cli @commitlint/config-conventional commitlint-config-gitmoji
```

---

## Step 6: Add Prepare Script

Check package.json for prepare script.

If not exists, add:
```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

If exists, append:
```json
{
  "scripts": {
    "prepare": "existing-command && husky"
  }
}
```

---

## Step 7: Initialize Husky

```bash
mkdir -p .husky

# Run prepare
pnpm prepare  # or npm/yarn
```

---

## Step 8: Create Husky Hooks

### commit-msg hook:
```bash
# .husky/commit-msg
pnpm commitlint --edit "$1"
```

For npm: `npx commitlint --edit "$1"`
For yarn: `yarn commitlint --edit "$1"`

### prepare-commit-msg hook:
```bash
# .husky/prepare-commit-msg
MSG_FILE=$1
MSG=$(cat "$MSG_FILE")

# Skip if gitmoji already exists
if ! echo "$MSG" | grep -qE '^:[a-z_]+:'; then
  if echo "$MSG" | grep -q '^feat'; then
    sed -i '' '1s/^/:sparkles: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^fix'; then
    sed -i '' '1s/^/:bug: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^docs'; then
    sed -i '' '1s/^/:page_facing_up: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^style'; then
    sed -i '' '1s/^/:art: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^refactor'; then
    sed -i '' '1s/^/:package: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^perf'; then
    sed -i '' '1s/^/:rocket: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^test'; then
    sed -i '' '1s/^/:rotating_light: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^build'; then
    sed -i '' '1s/^/:hammer: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^ci'; then
    sed -i '' '1s/^/:wrench: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^chore'; then
    sed -i '' '1s/^/:memo: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^revert'; then
    sed -i '' '1s/^/:wastebasket: /' "$MSG_FILE"
  elif echo "$MSG" | grep -q '^init'; then
    sed -i '' '1s/^/:tada: /' "$MSG_FILE"
  fi
fi
```

Make hooks executable:
```bash
chmod +x .husky/commit-msg .husky/prepare-commit-msg
```

---

## Step 9: Setup Pre-commit Hook

Check for linting tools:
```bash
grep -q '"typescript"' package.json && echo "typescript"
grep -q '"eslint"' package.json && echo "eslint"
grep -q '"lint-staged"' package.json && echo "lint-staged"
```

### All tools found (TypeScript + ESLint + lint-staged):
Auto-create pre-commit hook:
```bash
# .husky/pre-commit
echo "🔍 Running type check..."
pnpm type-check

echo "🧹 Running lint-staged..."
pnpm lint-staged
```

```
✅ Pre-commit hook added (type-check + lint-staged)
```

### Partial tools found:
Show detected tools:
```
📦 Detected:
  - TypeScript: ✓/✗
  - ESLint: ✓/✗
  - lint-staged: ✓/✗
```

**Ask user (AskUserQuestion):** "Add pre-commit hook with available tools?"

| Option | Description |
|--------|-------------|
| Yes | Add with detected tools |
| No | Skip pre-commit hook |

### No tools found:
**Ask user (AskUserQuestion):** "No linting tools detected. Add empty pre-commit hook?"

| Option | Description |
|--------|-------------|
| Yes | Create template for manual config |
| No | Skip pre-commit hook |

If Yes:
```bash
# .husky/pre-commit
# Add your pre-commit commands here
# Example: pnpm lint && pnpm test
```

---

## Step 10: Create Commitlint Config

Create `commitlint.config.cjs`:
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional', 'gitmoji'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert',
        'init',
      ],
    ],
    'subject-case': [0],
    'subject-empty': [0],
    'type-empty': [0],
    'start-with-gitmoji': [2, 'always'],
  },
};
```

---

## Step 11: Initial Commit

Check for existing commits:
```bash
git rev-list --count HEAD 2>/dev/null || echo "0"
```

If count is 0 (no commits):

**Ask user (AskUserQuestion):** "Create initial commit?"

| Option | Description |
|--------|-------------|
| Yes | Run /git:commit for initial commit |
| No | Skip, I'll commit manually |

### Yes selected:
```
📝 Launching /git:commit for initial commit...

Hint: Commit type → init
```

→ Execute /git:commit (the commit command will detect initial commit and suggest `init` type)

### No selected:
```
⏭️ Skipping initial commit.
   When ready: /git:commit
```

---

## Output Format

### On Success:
```
✅ Git initialization complete!

Repository:
  - Git initialized: ✓
  - Remote: origin → {url} (or "none")
  - .gitignore: ✓ ({type} template)

Commit tooling:
  - husky (git hooks)
  - commitlint (message linting)
  - gitmoji (auto emoji)

Hooks:
  - prepare-commit-msg: ✓
  - commit-msg: ✓
  - pre-commit: ✓/✗

Initial commit: ✓/pending

Next steps:
  - git push -u origin main (if remote configured)
  - /git:commit for your next commit
```

### On Failure:
```
❌ Setup failed: {error}

Please check:
  - Git is installed
  - Write permissions in project directory
  - Network connection (for gh CLI)
```

---

## Commit Type to Emoji Mapping

| Type | Gitmoji | Emoji |
|------|---------|-------|
| feat | :sparkles: | ✨ |
| fix | :bug: | 🐛 |
| docs | :page_facing_up: | 📄 |
| style | :art: | 🎨 |
| refactor | :package: | 📦 |
| perf | :rocket: | 🚀 |
| test | :rotating_light: | 🚨 |
| build | :hammer: | 🔨 |
| ci | :wrench: | 🔧 |
| chore | :memo: | 📝 |
| revert | :wastebasket: | 🗑 |
| init | :tada: | 🎉 |

---

## Constraints

1. **Preserve existing**: Don't overwrite without asking
2. **Respect package manager**: Use detected package manager
3. **macOS compatible**: sed uses `sed -i ''` syntax
4. **Non-destructive**: Always ask before replacing
5. **Workflow connection**: Link to /git:commit for initial commit
