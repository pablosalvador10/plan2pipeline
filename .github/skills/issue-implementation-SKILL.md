---
name: issue-implementation
description: >-
  Pick up an agent:ready issue and deliver a complete pull request. Use when
  starting work on a new issue, implementing a feature, or following the
  standard agent workflow. Triggers: implement issue, start task, agent:ready,
  pick up issue, new feature.
argument-hint: Provide the issue number to implement
---

## Purpose

End-to-end workflow for implementing a GitHub issue from reading the
specification to opening a reviewed pull request.

## When to Use

- An issue is labeled `agent:ready` and assigned to you
- You're starting work on a new task from the pipeline
- The auto-dispatch workflow triggers the coding agent

## Flow

1. **Read the full issue body** — understand Problem, Requirements, Acceptance
   Criteria, Constraints, and File hints
2. **Read project context:**
   - `.github/copilot-instructions.md` — coding guidelines
   - `.github/instructions/system-architecture.instructions.md` — architecture
   - Referenced files in `.github/specs/` — domain specifications
3. **Create a feature branch:** `git checkout -b issue-<N>-<short-slug>`
4. **Plan before coding** — identify all files to create/modify, consider
   impact on existing code, check for reusable patterns
5. **Implement in small, atomic commits:**
   - Production code first
   - Tests second (target 90%+ coverage on new code)
   - Documentation updates last
6. **Run all verification commands:**
   ```bash
   uv run pytest --tb=short -q
   uvx pre-commit run --all-files
   ```
7. **Open a draft PR** using the PR template (`.github/pull_request_template.md`)
   - Title: conventional commit format
   - Body: reference `Fixes #<N>`, fill KPI table, testing checklist
8. **Self-review** — re-read the Acceptance Criteria and verify every checkbox

## Guardrails

- **One issue per PR** — do not combine unrelated changes
- **Under 400 lines changed** — split larger tasks
- **Never modify** `.github/workflows/**`, `infra/**`, or `*.lock` files
- **Stage specifically** — use `git add <file>`, never `git add .`
