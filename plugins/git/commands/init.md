---
description: Initialize husky, commitlint, and gitmoji for conventional commits
---

# Initialize Git Commit Tooling

Set up husky, commitlint, and gitmoji for conventional commits with automatic emoji prefixes.

## Step 0: Check Project Type

```bash
# Check if this is a Node.js project
ls package.json 2>/dev/null
```

### If NO package.json found:

Detect project type and show alternatives:

```bash
# Python
ls requirements.txt pyproject.toml setup.py 2>/dev/null

# Go
ls go.mod 2>/dev/null

# Rust
ls Cargo.toml 2>/dev/null

# Ruby
ls Gemfile 2>/dev/null
```

Show message based on detected language:

```
⚠️ This is not a Node.js project.

/git:init is designed for Node.js projects (husky + commitlint).

Alternatives for your project:
```

| Language | Tool | Install |
|----------|------|---------|
| Python | pre-commit | `pip install pre-commit && pre-commit install` |
| Go | golangci-lint | `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest` |
| Rust | cargo-husky | `cargo add --dev cargo-husky` |
| Ruby | overcommit | `gem install overcommit && overcommit --install` |
| Other | git hooks | Manual setup in `.git/hooks/` |

**Exit after showing alternatives.**

---

## Step 1: Check Existing Setup

```bash
# Check if husky is already installed
ls .husky 2>/dev/null

# Check if commitlint config exists
ls commitlint.config.* .commitlintrc* 2>/dev/null
```

### If already set up:
Show current configuration status and ask user:

```
⚠️ Existing setup detected:
  - .husky/ directory: ✓ exists
  - commitlint config: ✓ exists
  - prepare-commit-msg: ✓ exists
```

**Ask user (AskUserQuestion):** "How would you like to proceed?"

| Option | Description |
|--------|-------------|
| Skip | Cancel and keep existing config |
| Merge | Add missing parts only (safe) |
| Overwrite | Replace all config (destructive) |

#### If "Skip": Exit with message "Keeping existing configuration."
#### If "Merge": Only add hooks/config that don't exist
#### If "Overwrite": Replace everything

## Step 2: Check Package Manager

Detect which package manager is being used:

```bash
# Check for lock files
ls pnpm-lock.yaml 2>/dev/null && echo "pnpm"
ls yarn.lock 2>/dev/null && echo "yarn"
ls package-lock.json 2>/dev/null && echo "npm"
```

If no lock file found:

**Ask user (AskUserQuestion):** "Which package manager do you use?"

| Option | Description |
|--------|-------------|
| pnpm | Use pnpm |
| npm | Use npm |
| yarn | Use yarn |

## Step 3: Install Dependencies

Based on package manager, install required dependencies:

### pnpm:
```bash
pnpm add -D husky @commitlint/cli @commitlint/config-conventional commitlint commitlint-config-gitmoji gitmoji-cli
```

### npm:
```bash
npm install -D husky @commitlint/cli @commitlint/config-conventional commitlint commitlint-config-gitmoji gitmoji-cli
```

### yarn:
```bash
yarn add -D husky @commitlint/cli @commitlint/config-conventional commitlint commitlint-config-gitmoji gitmoji-cli
```

## Step 4: Add Prepare Script

Check if `package.json` has a `prepare` script. If not, add it:

```json
{
  "scripts": {
    "prepare": "husky"
  }
}
```

If `prepare` script already exists, append husky:
```json
{
  "scripts": {
    "prepare": "existing-command && husky"
  }
}
```

## Step 5: Initialize Husky

```bash
# Create .husky directory
mkdir -p .husky

# Run prepare script to initialize husky
pnpm prepare  # or npm/yarn based on package manager
```

## Step 6: Create Husky Hooks

### Create commit-msg hook:
```bash
# .husky/commit-msg
pnpm commitlint --edit "$1"
```

For npm: `npx commitlint --edit "$1"`
For yarn: `yarn commitlint --edit "$1"`

### Create prepare-commit-msg hook:
```bash
# .husky/prepare-commit-msg
MSG_FILE=$1
MSG=$(cat "$MSG_FILE")

# Skip if gitmoji code already exists
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

## Step 7: Setup Pre-commit Hook (Auto-detect)

Check package.json for linting/type-checking tools:

```bash
# Check for TypeScript
grep -q '"typescript"' package.json && echo "typescript"

# Check for ESLint
grep -q '"eslint"' package.json && echo "eslint"

# Check for lint-staged
grep -q '"lint-staged"' package.json && echo "lint-staged"
```

### Case A: TypeScript + ESLint + lint-staged found
Automatically create pre-commit hook:

```bash
# .husky/pre-commit
echo "🔍 Running type check..."
pnpm type-check  # or npm run type-check / yarn type-check

echo "🧹 Running lint-staged..."
pnpm lint-staged
```

Show message: "✅ Pre-commit hook added (type-check + lint-staged)"

### Case B: Some tools found (partial)
Show what was detected and ask:

```
📦 Detected tools:
  - TypeScript: ✓
  - ESLint: ✓
  - lint-staged: ✗
```

**Ask user (AskUserQuestion):** "Add pre-commit hook with available tools?"

| Option | Description |
|--------|-------------|
| Yes | Add with detected tools |
| No | Skip pre-commit hook |

### Case C: No tools found

**Ask user (AskUserQuestion):** "No linting tools detected. Add pre-commit hook?"

| Option | Description |
|--------|-------------|
| Yes, manual | I'll configure it manually |
| No | Skip pre-commit hook |

If "Yes, manual": Create empty pre-commit hook template:
```bash
# .husky/pre-commit
# Add your pre-commit commands here
# Example: pnpm lint && pnpm test
```

## Step 8: Create Commitlint Config

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

## Step 9: Update .gitignore

Check if `.gitignore` includes `node_modules/`. If not, add:

```
# Dependencies
node_modules/
```

## Output Format

### On Success:
```
✅ Git commit tooling initialized!

Installed:
  - husky (git hooks)
  - commitlint (commit message linting)
  - gitmoji (automatic emoji prefixes)

Hooks configured:
  - prepare-commit-msg: Auto-add gitmoji ✓
  - commit-msg: Validate commit format ✓
  - pre-commit: Type-check + lint-staged ✓  (if detected)

Your commits will now:
  1. Run pre-commit checks (if configured)
  2. Auto-add gitmoji based on commit type
  3. Validate commit message format

Example:
  git commit -m "feat: Add new feature"
  → :sparkles: feat: Add new feature

Try it with /git:commit!
```

### On Failure:
```
❌ Setup failed: <error message>

Please check:
  - package.json exists
  - Git repository is initialized
  - Write permissions in project directory
```

## Commit Type to Emoji Mapping

| Type | Gitmoji Code | Emoji |
|------|--------------|-------|
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

## Constraints

1. **Preserve existing scripts**: Don't overwrite existing prepare script, append to it
2. **Respect package manager**: Use the same package manager as the project
3. **macOS compatible**: sed commands use macOS syntax (`sed -i ''`)
4. **Non-destructive**: Ask before overwriting existing config
