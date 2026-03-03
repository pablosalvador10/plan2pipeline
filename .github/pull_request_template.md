## Summary

<!-- Briefly describe what this PR does and why. -->

## Related Issue

<!-- Link the issue this PR addresses. Use "Fixes #N" to auto-close. -->

Fixes #

## Changes

<!-- List the key changes made in this PR. -->

- [ ] Change 1
- [ ] Change 2

## KPI Verification

<!-- Copy each KPI from the linked issue's KPI Contract section.
     For each KPI, document whether it is met and how you verified it.
     Do NOT remove any KPIs — mark as DEFERRED if not yet verifiable. -->

| KPI | Target | Baseline | Data Sufficiency | Status | Verification Method | Evidence |
|-----|--------|----------|-----------------|--------|---------------------|----------|
| _Copy from issue KPI Contract_ | _target_ | _baseline_ | YES/PARTIAL/NO | PASS / FAIL / DEFERRED | _from KPI Contract_ | _test name, file:line, or explanation_ |

## Testing

<!-- Describe how you tested these changes. -->

- [ ] All existing tests pass (`uv run pytest --tb=short -q`)
- [ ] New tests cover added behaviour
- [ ] Pre-commit checks pass (`uvx pre-commit run --all-files`)

## Checklist

- [ ] Code follows project conventions (see `.github/copilot-instructions.md`)
- [ ] All KPIs from the linked issue are addressed above
- [ ] Commit messages use conventional format (`feat:`, `fix:`, `docs:`, `chore:`)
- [ ] No unrelated changes included
- [ ] PR title follows conventional commit format
