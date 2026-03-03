---
name: ci-repair-playbook
description: >-
  Diagnose and fix CI check failures on pull requests. Use when a CI check
  fails, a build breaks, tests fail, or lint errors appear on a Copilot PR.
  Triggers: CI failure, build error, test failure, lint error, check failed.
argument-hint: Provide the PR number or link to the failing CI run
---

## Purpose

Step-by-step guide for diagnosing CI failures and applying targeted fixes
without introducing regressions.

## When to Use

- A CI check (test, lint, build, typecheck) fails on a PR
- The `ci-repair` label is applied to an issue
- The ci-repair-router workflow dispatches a repair cycle
- You see `check_suite.completed` with `conclusion: failure`

## Flow

1. **Read the failure log** — identify the exact error message and file/line
2. **Classify the failure type:**
   - `lint` — formatting or style violation → run the formatter/linter fix command
   - `test` — assertion failure → read the test, understand expected vs actual, fix the code or test
   - `build` — compilation/import error → fix missing imports, syntax, or type errors
   - `dependency` — missing package → check `pyproject.toml` / `package.json`, do NOT add packages without approval
3. **Apply the minimal fix** — change only what's needed to resolve the failure
4. **Run verification locally:**
   ```bash
   uv run pytest --tb=short -q          # Python tests
   uvx pre-commit run --all-files        # Lint + format
   pnpm test && pnpm lint               # TypeScript (if applicable)
   ```
5. **Commit with conventional format:** `fix: resolve <failure-type> — <brief description> (Fixes #N)`
6. **Push and verify** — confirm the CI check passes on the updated PR

## Guardrails

- **Max 2 repair attempts** per PR before escalating to `needs-human`
- **Never modify workflow files** (`.github/workflows/**`) during repair
- **Never add new dependencies** without explicit approval
- **Never skip or disable tests** to make CI pass
