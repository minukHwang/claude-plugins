---
description: Update README.md based on project changes
---

# Update README

Analyze project changes and update README.md accordingly.

## Step 0: README Existence Check

```bash
ls README.md 2>/dev/null
```

If README.md doesn't exist:
**Ask user:**
"README.md not found. Create one?"

| Option | Description |
|--------|-------------|
| Yes | Run `/readme:init` first |
| No | Stop |

## Step 1: Analysis Target Selection (Standalone Mode)

**Ask user:**
"What changes to reflect?"

| Option | Description |
|--------|-------------|
| 1 | Full project scan |
| 2 | PR changes (main..HEAD) |
| 3 | Describe manually |

※ When triggered from `/git:pr`: Auto-analyze PR changes (reuse conversation context)

## Step 2: Deep Analysis (`/git:pr` Step 5 Structure)

### Option 1: Full Project Scan

| Step | Analysis |
|------|----------|
| 2.1 | Project structure scan (`ls`, `package.json`, etc.) |
| 2.2 | Read key files with Read tool |
| 2.3 | Identify architecture/patterns |

### Option 2: PR Changes (Default for git integration)

| Step | Analysis |
|------|----------|
| 2.1 Commit Analysis | `git log origin/<TARGET>..HEAD --pretty=format:"%s%n%b"` |
| 2.2 Diff Analysis | `git diff origin/<TARGET>..HEAD` |
| 2.3 Changed Files | `git diff origin/<TARGET>..HEAD --name-only` |
| 2.4 **File Content** | **Read changed files with Read tool** |
| 2.5 Context Analysis | Conversation context (reuse if from git integration) |
| 2.6 Combined Analysis | Synthesize all info for README update points |

### Option 3: Manual Description

User describes changes → Claude analyzes relevant files

### Analysis Output

- New features/components identified
- Architecture changes detected
- Technical challenges identified
- README sections needing updates

## Step 3: Current README Analysis

```bash
cat README.md
```

Parse existing structure:
- Identify all sections
- Note placeholder content (`🚧`)
- Check for "In Development" markers

## Step 4: Change Detection

### 4.1 Version Change

| Check | Action |
|-------|--------|
| README has "In Development" | Check if package.json version >= 1.0.0 |
| Version upgraded to 1.0.0+ | Suggest removing "In Development" |

### 4.2 Screenshot Change

| Check | Action |
|-------|--------|
| README has `🚧 Screenshots are being prepared...` | Check if screenshots/ folder has images |
| Images found | Suggest adding screenshots |

### 4.3 Placeholder Detection

Find all placeholders:
- `🚧 Features are being developed...`
- `🚧 Technical challenges...`
- `🚧 API documentation coming soon...`

Check if content now exists for these sections.

### Change Notifications

```
🎉 Version is now 1.0.0+. Remove 'In Development' warning?
📸 Screenshots detected. Add them to README?
✨ New features detected. Update Features section?
🔧 Technical challenge solved. Document it?
```

## Step 5: Update Suggestions

Based on analysis, suggest specific updates:

**Ask user:**
"Update these sections?"

| Section | Suggested Change |
|---------|------------------|
| Features | "Add: User authentication with OAuth" |
| Tech Stack | "Add: React Query for server state" |
| Technical Challenges | "Add: Session refresh handling" |
| Screenshots | "Add: 3 new screenshots" |
| Status | "Remove 'In Development' warning" |

User can:
- Select all
- Select specific sections
- Skip

## Step 6: Execute Updates

For each approved update:

### Feature Updates
```markdown
## Key Features

- **Existing Feature**: Description
- **New Feature**: [Generated description based on analysis]
```

### Tech Stack Updates
Add new dependencies detected from package.json or imports

### Technical Challenges Updates
```markdown
## Technical Challenges & Solutions

### [New Challenge Title]
**Problem:** [Detected from code/conversation]
**Solution:** [How it was solved]
```

### Screenshot Updates
```markdown
## Screenshots

![Feature Name](screenshots/feature.png)
```

### Status Updates
Remove the `## ⚠️ Development Status` section

Use Edit tool for each modification

## Step 7: Verification

After updates:
```bash
cat README.md | head -50
```

Show summary of changes made.

## Output Format

### On Success:
```
✓ README.md updated successfully!

Changes made:
- Features: Added 2 new features
- Tech Stack: Added React Query
- Screenshots: Added 3 images
- Status: Removed 'In Development' warning

View changes: git diff README.md
```

### On No Changes:
```
ℹ️ No updates needed. README is up to date.
```

### On Failure:
```
✗ Failed to update README: <error message>
```

## Constraints

1. **Preserve structure**: Don't reorganize existing sections
2. **Additive updates**: Add content, don't remove unless requested
3. **User confirmation**: Always confirm before making changes
4. **English only**: All content in English
5. **Placeholder replacement**: Replace placeholders with real content when available
6. **Git context**: When from `/git:pr`, reuse analysis context to save tokens
