# Analyze Issue

Analyze GitHub Issues to determine their validity, priority, repository boundary responsibility, and recommended actions.

**Repository**: https://github.com/ZhuLinsen/daily_stock_analysis/issues

## Usage

```text
/analyze-issue <issue_number>
```

## Instructions

Analysis uses concise English, prioritizing the repository root `AGENTS.md`.

### Step 1: Fetch Issue Information

```bash
gh issue view <issue_number> --repo ZhuLinsen/daily_stock_analysis
gh issue view <issue_number> --repo ZhuLinsen/daily_stock_analysis --comments
```

If it's a bug, prioritize checking if the issue template provides:

- Whether synced to latest version
- Commit hash / version baseline
- Runtime environment and reproduction steps
- Logs or error messages

### Step 2: Answer 4 Core Questions

1. Is version clearly specified?
2. Is the issue real and verifiable?
3. Does it fall within repository responsibility boundary?
4. Is it worth immediate attention?

### Step 3: Combine Repository State for Evidence Check

- Read relevant code, configuration, tests, scripts, workflows, and documentation
- If issue involves API, data source fallback, report generation, notification sending, authentication, desktop app, or release process, clearly state impact scope
- Determine if actual bug, environment configuration issue, usage issue, or external dependency issue
- If suspected already fixed, check current code rather than just reading issue description

### Step 4: Form Conclusion

Provide at minimum these fields:

- `Version Baseline`: Latest / Not Latest / Not Provided
- `Is Valid`: Yes/No + Reason
- `Responsibility Boundary`: Yes/No + Which module
- `Priority`: P0/P1/P2/P3
- `Recommended Action`: Fix / Duplicate / Information Needed / Not a Bug / External Dependency

### Step 5: Save Analysis Result

Save analysis to `.claude/reviews/issues/issue-<number>.md` following this structure:

```markdown
# Issue Analysis: <Issue Title>

**Issue Number**: #<number>
**Status**: <Analyzing/Valid/Invalid/Duplicate>
**Priority**: P0/P1/P2/P3
**Date**: <YYYY-MM-DD>

## Quick Summary
<One sentence describing the issue and recommendation>

## Version & Environment
- Version Baseline: Latest / Not Latest / Not Provided
- Current version: <version>
- Environment details: <env details from issue>

## Validity Assessment
- Is it real: Yes/No
- Reproducibility: <details>
- Root cause hypothesis: <suspected cause>

## Boundary Assessment
- Within scope: Yes/No
- Relevant modules: <list modules>
- External dependencies: <if any>

## Risk & Impact
- User impact: <High/Medium/Low>
- System impact: <High/Medium/Low>
- Breaking change: <Yes/No>

## Recommended Action
- Action: <Fix/Duplicate/Information Needed/Not a Bug>
- Fix complexity: <Simple/Medium/Complex>
- Estimated effort: <hours>

## Related Issues/PRs
- Duplicates: <list>
- Related: <list>

## Evidence
- Code references: <links to relevant code>
- Similar issues: <links>
- Discussion: <summary of findings>

## Next Steps
1. <action item>
2. <action item>
```
