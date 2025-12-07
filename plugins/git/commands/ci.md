---
description: Monitor GitHub Actions CI status and analyze failures
---

# Monitor GitHub Actions CI

Check the CI/CD status for the current branch or PR, and analyze failures if any.

## Step 0: Check Current Context

```bash
# Get current branch
git branch --show-current

# Check if there's a PR for this branch
gh pr view --json number,title,url,statusCheckRollup 2>/dev/null
```

## Step 1: Get CI Status

### Option A: If PR exists
```bash
# Get PR check status
gh pr checks
```

### Option B: If no PR (just branch)
```bash
# Get recent workflow runs for current branch
gh run list --branch $(git branch --show-current) --limit 5
```

## Step 2: Display Status Summary

Show a summary table:

```
📊 CI Status for: <branch-name>

| Check | Status | Duration |
|-------|--------|----------|
| build | ✓ Pass | 2m 30s |
| test  | ✗ Fail | 1m 45s |
| lint  | ✓ Pass | 45s |

Overall: ✗ Failed (1/3 checks failed)
```

Status icons:
- ✓ Pass (green)
- ✗ Fail (red)
- ◯ Pending (yellow)
- ⊘ Skipped (gray)

## Step 3: On Failure - Analyze Logs

If any check failed:

```bash
# Get the failed run ID
gh run list --branch $(git branch --show-current) --status failure --limit 1 --json databaseId -q '.[0].databaseId'

# View failed run logs
gh run view <run-id> --log-failed
```

### Analyze Failure

1. **Identify the error type:**
   - Build error (compilation, bundling)
   - Test failure (unit, integration, e2e)
   - Lint error (ESLint, type-check)
   - Dependency issue (npm, pnpm install)
   - Environment issue (missing secrets, config)

2. **Extract key error messages:**
   - Look for `error:`, `Error:`, `failed`, `FAIL`
   - Get the specific file and line number if available
   - Capture the stack trace if present

3. **Provide actionable summary:**

```
❌ CI Failed: <check-name>

📍 Error Location:
   File: src/components/Button.tsx
   Line: 42

🔍 Error Message:
   Type 'string' is not assignable to type 'number'

💡 Suggested Fix:
   The `count` prop expects a number but received a string.
   Change: count="5" → count={5}
```

## Step 4: Offer Actions

After showing status:

**Ask user (AskUserQuestion):**
"What would you like to do?"

| Option | Description |
|--------|-------------|
| View logs | View detailed logs |
| Re-run | Re-run failed checks |
| Fix issues | Fix the issues (if failures found) |
| Done | Exit |

### If "View logs" selected:
```bash
gh run view <run-id> --log
```

### If "Re-run" selected:
```bash
gh run rerun <run-id> --failed
```

### If "Fix issues" selected:
- Read the relevant files mentioned in the error
- Analyze the code and propose fixes
- Apply fixes with user confirmation

## Step 5: Watch Mode (Optional)

If checks are still pending, offer watch mode:

"CI checks are still running. Watch for completion?"

If yes:
```bash
gh run watch
```

This will show real-time progress until all checks complete.

## Output Format

### All Passing:
```
✅ All CI checks passed!

| Check | Status | Duration |
|-------|--------|----------|
| build | ✓ Pass | 2m 30s |
| test  | ✓ Pass | 3m 15s |
| lint  | ✓ Pass | 45s |

Ready to merge! 🎉
```

### Some Failing:
```
❌ CI checks failed

| Check | Status | Duration |
|-------|--------|----------|
| build | ✓ Pass | 2m 30s |
| test  | ✗ Fail | 1m 45s |
| lint  | ✓ Pass | 45s |

Failed: test

📍 Error: src/utils/calc.test.ts:25
   Expected: 10
   Received: 9

💡 The calculation in `add()` function might be off by one.
```

### Pending:
```
⏳ CI checks in progress...

| Check | Status | Duration |
|-------|--------|----------|
| build | ✓ Pass | 2m 30s |
| test  | ◯ Running | 1m 20s |
| lint  | ◯ Queued | - |

Use watch mode to monitor progress.
```

## Constraints

1. **Use `gh` CLI**: All GitHub operations through `gh` command
2. **Current branch focus**: Default to current branch, no branch switching
3. **Non-destructive**: Only read/monitor, no code changes without user consent
4. **Clear error summary**: Always summarize errors in human-readable format
5. **Actionable suggestions**: Provide specific fix suggestions when possible
