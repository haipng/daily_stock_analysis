# Analyze PR

Analyze GitHub Pull Request to assess necessity, description completeness, verification evidence, major risks, and whether it can be directly merged.

**Repository**: https://github.com/ZhuLinsen/daily_stock_analysis/pulls

## Usage

```text
/analyze-pr <pr_number>
```

## Instructions

Analysis uses concise English, prioritizing repository root `AGENTS.md` and `.github/PULL_REQUEST_TEMPLATE.md`.

### Step 1: Fetch PR Basic Information

```bash
gh pr view <pr_number> --repo ZhuLinsen/daily_stock_analysis
gh pr view <pr_number> --repo ZhuLinsen/daily_stock_analysis --comments
gh pr checks <pr_number> --repo ZhuLinsen/daily_stock_analysis
gh pr diff <pr_number> --repo ZhuLinsen/daily_stock_analysis
```

If CI failed, prioritize checking failed logs rather than immediately re-running all checks locally:

```bash
gh run view <run_id> --log-failed
```

### Step 2: Check Title & Description Completeness

First check if PR title follows `AGENTS.md` non-blocking recommendation:

- Format should be `<type>: <change summary>`, e.g., `fix: Fix missing historical records in market review`
- Type should be `fix`/`feat`/`refactor`/`docs`/`chore`/`test`/`ci`
- Should not include `[codex]`, `codex`, `autocode`, `copilot`, or other tool/agent source prefixes
- Title should describe actual changes; if title doesn't match diff, note in description completeness, but shouldn't be sole review blocker

Against `.github/PULL_REQUEST_TEMPLATE.md`, confirm coverage of:

- `PR Type`
- `Background And Problem`
- `Scope Of Change`
- `Issue Link`
- `Verification Commands And Results`
- `Compatibility And Risk`
- `Rollback Plan`

### Step 3: Evidence-Based Review

Skim diff for scale and impact:

```bash
gh pr diff <pr_number> --stat
```

Read relevant code, configuration, tests, scripts, workflows, and documentation. Key areas:

- API/Schema changes: Check backward compat, client impact
- Data source fallback: Confirm resilience maintained
- Report generation: Verify output format unchanged
- Notification sending: Confirm delivery path
- Authentication: Verify security maintained
- Desktop/packaging: Check platform-specific impact
- Deployment: Verify staging/prod plan

### Step 4: CI & Verification Assessment

Priority order:

1. **Check CI results first**:
   - All green (passing): No need to re-run locally
   - Some failures: Examine failure logs `gh run view <run_id> --log-failed`
   - No CI: Recommend PR author to ensure CI coverage

2. **Assessment categories**:
   - Backend changes: Check `backend-gate`, `docker-build` status
   - Frontend changes: Check `web-gate` status
   - Docs-only: No code verification needed, but verify paths/commands accurate
   - Desktop changes: Check desktop build status

3. **Re-run locally only if**:
   - CI results conflict with change description
   - CI infrastructure suspect (timeouts, flakes)
   - Locally-specific environment issues (OS, Python version)

### Step 5: Merge Decision Matrix

| Condition | Action |
|-----------|--------|
| All CI green + description complete + no blocking issues | ✅ Ready to merge |
| Title/desc mismatch but content valid | ⚠️ Suggest improve title, not blocker |
| CI failures + description unclear | ❌ Request author fix + re-verify |
| Major untested impact + no CI coverage | ❌ Request test evidence |
| Backward-incompatible change undocumented | ❌ Request CHANGELOG + docs update |

### Step 6: Save Analysis Result

Save analysis to `.claude/reviews/pulls/pr-<number>.md`:

```markdown
# PR Review: <PR Title>

**PR Number**: #<number>
**Status**: <Ready/Needs Work/Blocked>
**Date**: <YYYY-MM-DD>

## Executive Summary
- **Recommendation**: <Approve/Request Changes/Close>
- **Priority**: <P0/P1/P2/P3>
- **Risk Level**: <Low/Medium/High>

## PR Meta
- Type: <fix/feat/refactor/docs/chore/test/ci>
- Title quality: <Good/Needs improvement/Wrong>
- Description completeness: <Complete/Partial/Missing>
- Author: <github username>

## Change Scope
- Files changed: <count>
- Lines +/- : <stats>
- Impact area: <backend/frontend/desktop/docs/workflow>

## CI Status
- Backend gate: <✅ Pass / ❌ Fail / ⏭️ Skipped>
- Web gate: <✅ Pass / ❌ Fail / ⏭️ Skipped>
- Docker build: <✅ Pass / ❌ Fail / ⏭️ Skipped>
- Other checks: <summary>

## Verification Evidence
- Described verification: <Yes/No>
- Commands run: <Yes/No/Not applicable>
- Local test: <Recommended/Not needed>
- Integration test: <Recommended/Not needed>

## Risk Assessment

### Breaking Changes
- API: <Yes/No> - <details>
- Schema: <Yes/No> - <details>
- Config: <Yes/No> - <details>
- Other: <Yes/No> - <details>

### Backward Compatibility
- Status: <Fully compatible/Partially compatible/Incompatible>
- Migration needed: <Yes/No>
- Deprecation path: <Yes/No>

### Data Safety
- Data migration: <None/Automatic/Manual required>
- Rollback: <Reversible/Requires manual recovery>

## Code Quality

### Correctness
- Logic: ✅ / ❌ - <issues>
- Error handling: ✅ / ❌ - <issues>
- Edge cases: ✅ / ❌ - <issues>

### Consistency
- Matches existing patterns: ✅ / ⚠️ / ❌
- Code style: ✅ / ⚠️ / ❌
- Documentation: ✅ / ⚠️ / ❌

## Merge Blockers
- CI failure: <Yes/No> - <if yes, describe>
- Missing verification: <Yes/No> - <if yes, describe>
- Design concern: <Yes/No> - <if yes, describe>
- Breaking change undocumented: <Yes/No>

## Suggested Improvements (Non-blocking)
1. <suggestion>
2. <suggestion>

## Approval Conditions
- [ ] All CI passing
- [ ] Description complete per template
- [ ] No merge blockers
- [ ] Verification results included

## Next Steps
1. <action item>
2. <action item>

## Reviewers Checklist
- [ ] Reviewed diff
- [ ] Checked CI results
- [ ] Read PR description
- [ ] Verified affected modules
- [ ] Assessed risk
- [ ] Ready to approve/request changes
```
