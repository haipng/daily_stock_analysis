# Fix Issue

Implement fix based on issue analysis results, and complete verification, risk, and rollback documentation per repository rules.

**Repository**: https://github.com/ZhuLinsen/daily_stock_analysis

## Usage

```text
/fix-issue <issue_number>
```

## Prerequisites

Preferably first complete `/analyze-issue <issue_number>` to ensure the issue is valid and boundaries are clear.

## Instructions

### Step 1: Confirm Analysis Baseline

Check if `.claude/reviews/issues/issue-<number>.md` exists; if not, either complete issue analysis first or include minimal analysis conclusions in this fix.

### Step 2: Choose Safe Work Method

- Default: Make minimal relevant changes based on current work tree
- Don't auto-execute `git pull`
- Don't auto-switch branches or alter user current state
- Only perform minimal necessary branch operations if user explicitly requests

### Step 3: Implement Fix

- Locate relevant files per issue conclusion
- Prioritize reusing existing modules, config entry points, scripts, and tests
- Maintain backward compatibility for default behavior; avoid breaking fallback/fail-open
- If fix involves user-visible behavior, config semantics, CLI/API, deployment, notifications, report structure, sync update relevant docs, `docs/CHANGELOG.md`, `.env.example`
- When writing to `docs/CHANGELOG.md`, append one line to `[Unreleased]` section formatted as `- [type] description`, where `[type]` is chosen from `[新功能]/[改进]/[修复]/[文档]/[测试]/[chore]` based on change nature; only use `[修复]` when fixing bugs; **do not** add `### Category Headers` inside `[Unreleased]`
- `README.md` only carries project positioning, core capabilities, quick start, main entry points, sponsorship/cooperation; don't update unless necessary to avoid bloat
- Detailed module behavior, page interaction, topic configuration, troubleshooting, field contracts, implementation semantics, edge cases - prioritize updating corresponding `docs/*.md`

### Step 4: Verify Per Impact Scope

Execute verification from `AGENTS.md` validation matrix closest to changes:

- Backend priority: `./scripts/ci_gate.sh`
- Minimum backend requirement: `python -m py_compile <changed_python_files>`
- Frontend: `cd apps/dsa-web && npm ci && npm run lint && npm run build`
- Desktop: Build Web first, then desktop

If unable to complete full verification, must record gaps, reasons, and potential risks.

### Step 5: Document Fix

Save to `.claude/reviews/issues/issue-<number>-fix.md`:

```markdown
# Fix Implementation: Issue #<number>

**Issue**: <Issue Title>
**Fix Date**: <YYYY-MM-DD>
**Status**: <In Progress/Ready for Review/Merged>

## What Changed
- <file1>: <change description>
- <file2>: <change description>

## Why This Fix
- Root cause: <root cause from analysis>
- Solution approach: <explain why this fix)>
- Alternative considered: <if any>

## Files Modified
- [src/module/file.py](../../src/module/file.py) - <what changed>
- [docs/guide.md](../../docs/guide.md) - <what changed>

## Verification Results
- Backend: ✅ `./scripts/ci_gate.sh` passed
- Frontend: ✅ Build succeeded
- Desktop: ✅ Build succeeded / ⚠️ Not tested (reason)

## Backward Compatibility
- Breaking: No
- Migration needed: No
- Fallback behavior: <describe>

## Risk Assessment
- Risk level: Low/Medium/High
- Data impact: None / <describe>
- Performance impact: None / <describe>

## Rollback Plan
```bash
# Rollback command
git revert <commit-hash>
```
- Steps: <detailed rollback steps>
- Data recovery: <if needed>
- Notification: <who to notify>

## Related PRs
- Will create: Yes/No
- Link: <if exists>

## Testing Evidence
- Local test: ✅ <test output>
- Unit tests: ✅ <pass rate>
- Integration: ✅ <tested>

## Next Steps
1. <action item>
2. <action item>
```
