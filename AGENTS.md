# AGENTS.md — plan2pipeline

> This file is read automatically by GitHub Copilot, OpenAI Codex, Cursor,
> and other AI coding agents before they start work.

## Finding Work

Issues labeled **`agent:ready`** have no unresolved dependencies and are
ready to be picked up.  To list them:

```bash
gh issue list --label "agent:ready" --state open --json number,title,labels
```

Each issue follows a structured template with these sections:
- **Problem** — what needs to change and why
- **Files & Scope** — which files to create or modify
- **Requirements** — numbered, measurable requirements
- **Acceptance Criteria** — checkboxes for "definition of done"
- **Constraints** — what NOT to do
- **Reference Patterns** — links to existing code or docs

**Tip:** Think of each issue body as a prompt — read the entire thing
before writing any code.

## Workflow

1. Pick an `agent:ready` issue
2. Create a branch: `git checkout -b issue-<number>-<slug>`
3. Read the full issue body, then consult any files in
   `.github/specs/` and `.github/instructions/` if they exist
4. Implement the requirements, respecting all constraints
5. Write or update tests and ensure they pass
6. Run all checks (see Verification below)
7. Commit with a descriptive message referencing the issue: `Fixes #<number>`
8. Push and open a PR using the PR template

## Git Best Practices

- **Stage specifically:** Use `git add <filename>` — never `git add .`
- **Atomic commits:** One logical change per commit
- **Rebase workflow:** `git fetch origin && git rebase origin/main`
  before pushing — do not create merge commits
- **Branch naming:** `issue-<number>-<short-slug>` (e.g. `issue-42-add-auth`)
- **Commit messages:** Use conventional format: `feat:`, `fix:`, `docs:`, `chore:`
  followed by a brief description and `Fixes #<number>`

## Verification — Run Before Every PR

```bash
# Install pre-commit hooks (run once after clone)
uvx pre-commit install

# Python checks
uv run pytest --tb=short -q
uvx pre-commit run --all-files

# TypeScript checks (if applicable)
pnpm test
pnpm lint
pnpm typecheck
```

All checks MUST pass before opening a PR.  If a check fails, fix the
issue and retry — do not skip checks or disable rules.

**IMPORTANT:** Always run `uvx pre-commit run --all-files` before every
commit.  Pre-commit hooks enforce formatting, linting, and folder structure
rules.  Commits that fail pre-commit checks will be rejected in CI.

## PR Guidelines

When opening a pull request:

1. Use the PR template in `.github/pull_request_template.md`
2. Fill in the summary, issue reference, and testing checklist
3. Ensure the PR title follows conventional commit format
4. One issue per PR — do not combine unrelated changes
5. Keep PRs small and focused (under 400 lines changed when possible)
6. If iterating after review, batch related fixes into one commit

## Labels

| Label | Meaning |
|-------|---------|
| `agent:ready` | No unresolved deps — ready to implement |
| `must-have` | P0 — required for MVP |
| `should-have` | P1 — important but deferrable |
| `nice-to-have` | P2 — low priority |
| `swe` / `business` / `datasci` | Domain that originated the issue |
| `feature` / `bug` / `infra` / `security` / `data` / `docs` | Work type |
| `risk` | Flagged risk requiring extra care |

## Effort Sizing

Each issue has an **Effort** badge at the top of its body:
- **S** — under 30 minutes
- **M** — 30–60 minutes
- **L** — 60–90 minutes
- **XL** — should have been split (flag for review)

## Dependencies

Issues with dependencies list them at the bottom of their body:

```
**Dependencies:**
- Depends on: #3 (Set up CI pipeline)
```

Do NOT start an issue until all its dependencies are resolved (merged).
When a dependency is resolved, the issue receives the `agent:ready`
label automatically via the agent-monitor workflow.

## Custom Agents

This repository includes specialized agent profiles in `.github/agents/`:

| Agent | Purpose |
|-------|---------|
| `developer.agent.md` | Primary code implementer — picks `agent:ready` issues and delivers PRs |
| `test-specialist.agent.md` | Write and improve tests without modifying production code |
| `implementation-planner.agent.md` | Create implementation plans and technical specs |
| `qa-reviewer.agent.md` | Review PRs created by other agents for quality and correctness |
| `orchestrator.agent.md` | Fleet coordinator — monitors progress, unblocks stalled work |

To use a custom agent, select it from the agent dropdown in your IDE or
on GitHub.com when assigning work.

## Skills

Agent playbooks in `.github/skills/` provide step-by-step guides for
common pipeline tasks.  These are loaded on demand when the task matches
the skill's trigger keywords.

| Skill | When to Use |
|-------|-------------|
| `ci-repair-playbook` | CI check fails — diagnose and fix |
| `issue-implementation` | Starting work on an `agent:ready` issue |
| `pr-review-checklist` | Reviewing a pull request |
| `pipeline-debugging` | Pipeline stalled or watchdog alert fired |

## Model & Reasoning Configuration

When working on this project, AI coding agents SHOULD use:

- **Model**: Claude Opus 4.6 (or latest Opus) with extended thinking enabled
- **Reasoning**: Enable extended thinking / chain-of-thought for complex tasks
  (architecture decisions, multi-file refactors, debugging)
- **Temperature**: Use low temperature (0.1–0.3) for code generation,
  higher (0.5–0.7) for creative tasks like documentation

Extended thinking helps the agent:
- Understand the full impact of changes across the codebase
- Plan multi-step implementations before writing code
- Reason about edge cases and error handling
- Cross-reference specs and architecture docs effectively

## Project Context

Read these files before starting work (if they exist — some are
project-specific and may not be present in a bare template clone):

| File | Purpose |
|------|---------|
| `.github/copilot-instructions.md` | Project-specific coding guidelines |
| `.github/instructions/system-architecture.instructions.md` | System design, data flows, component diagram |
| `.github/specs/swe-analysis.md` | Full SWE specification |
| `.github/specs/business-analysis.md` | Full Business specification |
| `.github/specs/datasci-analysis.md` | Full Data Science specification |
| `.github/pull_request_template.md` | PR template — fill this out when opening PRs |

## Repository Structure

### Template Files (ship with every repo)

These files are part of the template and are always present:

| Path | What It Contains |
|------|------------------|
| `AGENTS.md` | This file — cross-tool agent workflow |
| `README.md` | Repository overview with pipeline badge |
| `.github/pull_request_template.md` | Standardised PR format for consistent, reviewable PRs |
| `.github/automation-policy.yml` | Machine-readable automation boundary |
| `.github/prompts/repo-assist.md` | Implementation agent prompt |
| `.github/prompts/pr-review-agent.md` | Review agent prompt |
| `.github/workflows/auto-dispatch.yml` | Label-driven issue dispatch |
| `.github/workflows/auto-dispatch-requeue.yml` | Deferred issue requeue |
| `.github/workflows/repo-assist.yml` | Copilot implementation workflow |
| `.github/workflows/pr-review-agent.yml` | Copilot review workflow |
| `.github/workflows/pr-review-submit.yml` | Formal review gate |
| `.github/workflows/agent-monitor.yml` | Dependency unlock + mark-PR-ready |
| `.github/workflows/close-issues.yml` | Deterministic issue closing on merge |
| `.github/workflows/pipeline-status.yml` | Rolling status dashboard |
| `.github/workflows/ci-repair-router.yml` | Self-healing CI failure loop |
| `.github/workflows/pipeline-watchdog.yml` | 30-min cron health check |
| `.github/agents/developer.agent.md` | Primary code implementer agent profile |
| `.github/agents/test-specialist.agent.md` | Testing-focused agent profile |
| `.github/agents/implementation-planner.agent.md` | Planning agent profile |
| `.github/agents/qa-reviewer.agent.md` | QA review agent profile |
| `.github/agents/orchestrator.agent.md` | Fleet coordinator agent profile |
| `.github/skills/ci-repair-playbook-SKILL.md` | CI failure diagnosis playbook |
| `.github/skills/issue-implementation-SKILL.md` | Issue workflow playbook |
| `.github/skills/pr-review-checklist-SKILL.md` | Structured review playbook |
| `.github/skills/pipeline-debugging-SKILL.md` | Stall diagnosis playbook |

### Project-Specific Files (generated per project by HUGO)

These files are pushed by the orchestrator during full project bootstrapping.
They depend on the project's spec analyses and are NOT part of the template:

| Path | What It Contains |
|------|------------------|
| `.github/copilot-instructions.md` | Project-specific coding context (architecture, constraints, APIs) |
| `.github/instructions/system-architecture.instructions.md` | Component diagram, data flows, change impact matrix |
| `.github/specs/swe-analysis.md` | Full SWE analysis — architecture, ADRs, implementation plan |
| `.github/specs/business-analysis.md` | Full Business analysis — stakeholders, ROI, feasibility |
| `.github/specs/datasci-analysis.md` | Full Data Science analysis — metrics, evaluation, data reqs |
| `.github/workflows/copilot-setup-steps.yml` | Pre-installs dependencies so agents can build and test immediately |

Issues on this repository follow a structured template with Problem,
Files & Scope, Requirements, Acceptance Criteria, Constraints, and
Reference Patterns.  Start with issues labeled `agent:ready`.

Milestones group issues into implementation phases.  The project board
provides a Kanban view of all work items.

## Conventions

- **Python**: Follow `ruff` rules, use `structlog` for logging, async everywhere
- **TypeScript**: Follow ESLint config, strict TypeScript
- **Commits**: Conventional commits preferred (`feat:`, `fix:`, `docs:`, `chore:`)
- **PRs**: One issue per PR, reference the issue number, use the PR template
- **Tests**: Every PR must include tests for new behaviour
- **Pre-commit**: Always run `uvx pre-commit run --all-files` before committing
