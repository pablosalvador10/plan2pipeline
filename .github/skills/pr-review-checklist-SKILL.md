---
name: pr-review-checklist
description: >-
  Structured review flow for evaluating pull requests. Use when reviewing a PR,
  checking code quality, verifying acceptance criteria, or posting a review
  verdict. Triggers: review PR, check PR, code review, QA review, FDE-VERDICT.
argument-hint: Provide the PR number to review
---

## Purpose

Systematic checklist for reviewing pull requests created by coding agents,
ensuring correctness, test coverage, and compliance with project standards.

## When to Use

- A new PR is opened or updated by the coding agent
- The pr-review-agent workflow dispatches you
- Manual code review is requested

## Flow

1. **Read the linked issue** — understand what was requested
2. **Read the PR description** — verify it references the issue and uses the template
3. **Review the diff systematically:**
   - [ ] All Acceptance Criteria from the issue are met
   - [ ] Code follows project conventions (`.github/copilot-instructions.md`)
   - [ ] No unnecessary files changed
   - [ ] No sensitive paths modified (`.github/workflows/**`, `infra/**`)
   - [ ] Error handling is present for failure cases
   - [ ] No hardcoded secrets, tokens, or credentials
4. **Check test coverage:**
   - [ ] New behavior has corresponding tests
   - [ ] Edge cases are covered
   - [ ] Tests are meaningful (not just smoke tests)
   - [ ] Test names describe what they verify
5. **Check KPI compliance** (from the issue):
   - [ ] Each KPI has a measured value
   - [ ] No KPI is marked `FAIL`
6. **Post verdict** as a `[FDE-VERDICT]` comment:
   ```
   [FDE-VERDICT] APPROVE | REQUEST_CHANGES
   Summary: <one-line summary>
   KPIs: <pass/fail counts>
   ```

## Guardrails

- **Never approve a PR with failing CI checks**
- **Never approve modification of sensitive paths** without human override
- **Request changes** if any Acceptance Criteria checkbox is unmet
- **Do not review your own PRs** — the review agent must be a different agent
