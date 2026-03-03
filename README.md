# plan2pipeline

A GitHub template repository that provides the complete infrastructure for
running an autonomous spec-to-code pipeline.  Create a repository from this
template, push your project specification and task plan, and the pipeline
dispatches AI coding agents that implement every task as a pull request --
reviewed, tested, and merged without human intervention.

---

## Table of Contents

- [What This Template Does](#what-this-template-does)
- [Architecture Overview](#architecture-overview)
- [Pipeline Flow](#pipeline-flow)
- [Getting Started](#getting-started)
- [Repository Contents](#repository-contents)
  - [Workflows](#workflows)
  - [Agent Prompts](#agent-prompts)
  - [Agent Profiles](#agent-profiles)
  - [Policy and Documentation](#policy-and-documentation)
- [Labels](#labels)
- [Automation Policy](#automation-policy)
  - [Autonomous Actions](#autonomous-actions)
  - [Human-Required Actions](#human-required-actions)
  - [Guardrails](#guardrails)
- [Dynamic Files (Pushed Per Project)](#dynamic-files-pushed-per-project)
- [Configuration](#configuration)
  - [Kill Switches](#kill-switches)
  - [Concurrency and Limits](#concurrency-and-limits)
- [Agent Conventions](#agent-conventions)
  - [Branch Naming](#branch-naming)
  - [Commit Format](#commit-format)
  - [PR Requirements](#pr-requirements)
  - [Verification Commands](#verification-commands)
- [Frequently Asked Questions](#frequently-asked-questions)
- [License](#license)

---

## What This Template Does

This template contains **25 deterministic files** that together form a
self-sustaining autonomous development pipeline.  The pipeline:

1. Receives a set of GitHub issues, each describing a self-contained coding
   task with acceptance criteria, dependencies, and verification commands.
2. Dispatches issues to a coding agent (GitHub Copilot) via label-driven
   workflow triggers.
3. The coding agent reads the issue, creates a feature branch, implements the
   code, writes tests, and opens a pull request.
4. A review agent inspects every pull request for correctness, test coverage,
   KPI compliance, and security -- posting a structured verdict.
5. Approved pull requests are auto-merged.  On merge, linked issues close and
   dependent issues unlock automatically.
6. If CI fails, a self-healing loop re-dispatches the coding agent with
   failure context (up to 2 repair attempts).
7. A watchdog runs every 30 minutes to detect and recover stalled work.
8. A status dashboard issue updates every 15 minutes with live pipeline
   metrics.

No human intervention is required for the standard path.  Sensitive file
changes (workflows, infrastructure, secrets) are blocked and escalated for
human review.

---

## Architecture Overview

```
 Issue Created          Label: agent:ready           repo-assist.yml
 (with deps,    ──────> (auto-dispatch routes  ──────> (Copilot agent reads
  acceptance             to coding agent)               issue, implements
  criteria)                                             code, opens PR)
       │                                                     │
       │                                                     ▼
       │                pr-review-agent.yml            Draft PR opened
       │                (review agent inspects  <────── (auto-dispatched
       │                 diff, posts verdict)            on PR open)
       │                       │
       │                       ▼
       │                pr-review-submit.yml
       │                (formal review gate,
       │                 auto-merge on APPROVE)
       │                       │
       │                       ▼
       │                close-issues.yml              agent-monitor.yml
       │                (close linked issues  ──────> (unlock dependent
       │                 on merge)                     issues, re-label
       │                                               as agent:ready)
       │                                                     │
       │                                                     ▼
       │                                              Next issue dispatched
       │                                              (cycle repeats)
       │
       │  On CI failure:
       │  ci-repair-router.yml ──> re-dispatch with failure context (max 2x)
       │
       │  On stall (30 min no activity):
       │  pipeline-watchdog.yml ──> re-dispatch or escalate
       │
       └──> pipeline-status.yml ──> rolling dashboard issue (every 15 min)
```

---

## Pipeline Flow

The pipeline processes tasks in dependency order using a DAG (directed acyclic
graph) with milestones representing execution phases.

**Phase 1 -- Foundation:** Root tasks with no dependencies are dispatched
immediately.  They receive the `agent:ready` label and are picked up by the
auto-dispatch workflow.

**Phase 2+ -- Dependent Work:** Tasks with dependencies receive the
`agent:deferred` label.  When all dependencies close (via PR merge), the
agent-monitor workflow detects the resolution and re-labels the task as
`agent:ready`, triggering the next dispatch cycle.

**Concurrency:** Up to 5 agents run simultaneously.  When the limit is
reached, additional ready issues are deferred and re-queued when a slot
opens.

**Self-Healing:** If a CI check fails on a Copilot PR, the ci-repair-router
extracts the failure context (lint, test, build, or dependency error),
posts a structured repair command on the issue, and re-dispatches the
coding agent in `ci-repair` mode.  After 2 failed repair attempts, the
issue is escalated with a `needs-human` label.

---

## Getting Started

### Prerequisites

- A GitHub account with access to GitHub Copilot coding agent (Copilot
  Workspace or Copilot agent via `gh copilot` CLI).
- The `gh` CLI installed and authenticated.
- Repository-level secrets or environment configuration as needed by your
  project (none required by this template itself).

### Step 1: Create a Repository From This Template

Click **"Use this template"** on the GitHub UI, or use the CLI:

```bash
gh repo create my-org/my-project \
  --template pablosalvador10/plan2pipeline \
  --public
```

This copies all 25 automation files into your new repository.

### Step 2: Push Your Project-Specific Files

The template provides the automation skeleton.  Your project needs these
dynamic files pushed after creation:

| File | Purpose |
|------|---------|
| `.github/copilot-instructions.md` | Project-specific coding instructions for the agent |
| `.github/instructions/system-architecture.instructions.md` | Architecture reference document |
| `.github/specs/swe-analysis.md` | Software engineering analysis / specification |
| `.github/specs/task-plan.md` | Task breakdown with dependencies and acceptance criteria |
| `.github/copilot-setup-steps.yml` | Environment setup for the coding agent (language, deps) |

### Step 3: Create GitHub Issues

Create one issue per task.  Each issue should include:

- A clear title with the task ID and role (e.g., `[T1-store] [swe] Implement JSON storage module`)
- Acceptance criteria as a checklist
- Dependencies listed as `- Depends on: #N` (referencing other issue numbers)
- Verification commands
- File hints

Label root tasks (no dependencies) with `agent:ready` and `pipeline`.
Label dependent tasks with `agent:deferred` and `pipeline`.

The auto-dispatch workflow picks up `agent:ready` issues automatically.

### Step 4: Monitor

- Watch the **Pipeline Status** issue (created automatically) for live
  metrics and progress.
- Review PRs opened by the coding agent.
- Issues labeled `needs-human` require manual intervention.

---

## Repository Contents

### Workflows

| File | Trigger | Purpose |
|------|---------|---------|
| `.github/workflows/auto-dispatch.yml` | `issues.labeled` (`agent:ready`) | Routes ready issues to the coding agent.  Validates the issue has a `pipeline` label, checks concurrency (max 5), defers overflow. |
| `.github/workflows/auto-dispatch-requeue.yml` | `workflow_run.completed` (repo-assist) | Re-queues `agent:deferred` issues when a slot opens.  FIFO order. |
| `.github/workflows/repo-assist.yml` | `workflow_dispatch` | Core execution engine.  Installs Copilot CLI, reads the issue + prompt, implements code, opens a PR.  Supports `implement`, `ci-repair`, and `command` modes. |
| `.github/workflows/pr-review-agent.yml` | `pull_request` (opened, reopened, ready_for_review) | Dispatches the review agent on Copilot PRs.  Skips QA-authored PRs to prevent review loops. |
| `.github/workflows/pr-review-submit.yml` | `issue_comment.created` (`[FDE-VERDICT]`) | Identity-separation gate.  Parses the review verdict, checks sensitive paths against automation policy, submits a formal GitHub review from `github-actions[bot]`, and queues auto-merge. |
| `.github/workflows/agent-monitor.yml` | `issues.closed` + 5-min cron | Unlocks dependent issues when all their dependencies close.  Also undrafts finished Copilot PRs. |
| `.github/workflows/close-issues.yml` | `pull_request.closed` (merged) | Defense-in-depth issue closing.  Extracts `Fixes/Closes/Resolves #N` from PR body and closes linked issues. |
| `.github/workflows/pipeline-status.yml` | 15-min cron | Creates and updates a rolling `[FDE] Pipeline Status` issue with live metrics: progress bar, completion counts, stalled issue detection. |
| `.github/workflows/ci-repair-router.yml` | `check_suite.completed`, `workflow_run.completed` | Self-healing CI loop.  Identifies failing Copilot PRs, classifies failures (lint/test/build/dependency), posts repair commands, re-dispatches the coding agent.  Max 2 attempts before escalation. |
| `.github/workflows/pipeline-watchdog.yml` | 30-min cron | Stall detection.  Re-dispatches issues stuck in `agent:in-progress` for more than 30 minutes.  Detects orphan Copilot PRs.  Max 2 retries before escalation. |

### Agent Prompts

| File | Purpose |
|------|---------|
| `.github/prompts/repo-assist.md` | Implementation agent prompt.  Instructs the coding agent to read the issue, create a feature branch (`issue-{N}-{slug}`), implement code and tests, run verification, and open a PR with `Fixes #N`. |
| `.github/prompts/pr-review-agent.md` | Review agent prompt.  Instructs the review agent to inspect the PR diff, verify requirements and KPI compliance, check sensitive paths against automation policy, and post a structured `[FDE-VERDICT]` comment. |

### Agent Profiles

Agent profiles define the capabilities and constraints of each specialized
agent.  They are referenced by name in issue dispatch and workflow routing.

| File | Role | Description |
|------|------|-------------|
| `.github/agents/developer.agent.md` | Primary implementer | Picks `agent:ready` issues, writes production code + tests + documentation in a single PR.  Follows a 5-phase strategy: Understand, Plan, Implement, Verify, Open PR.  Targets 90%+ test coverage on new code. |
| `.github/agents/test-specialist.agent.md` | Test specialist | Writes and improves tests for existing production code without modifying it.  Focuses on edge cases, coverage gaps, and regression tests. |
| `.github/agents/implementation-planner.agent.md` | Implementation planner | Creates implementation plans and technical specs from high-level requirements.  Breaks down work into actionable tasks. |
| `.github/agents/qa-reviewer.agent.md` | Quality reviewer | Reviews every PR for correctness, acceptance criteria, KPI compliance, test coverage, security, and performance.  Posts a `[FDE-VERDICT]` comment with `APPROVE` or `REQUEST_CHANGES`.  Will not approve if any KPI is `FAIL`. |
| `.github/agents/orchestrator.agent.md` | Fleet coordinator | Monitors pipeline health, unblocks stalled work, coordinates handoffs between agents, and escalates when needed. |

### Skills

Agent playbooks in `.github/skills/` provide step-by-step guides for
common pipeline tasks:

| File | Purpose |
|------|---------||
| `.github/skills/ci-repair-playbook-SKILL.md` | Diagnose and fix CI failures — lint, test, build, and dependency errors |
| `.github/skills/issue-implementation-SKILL.md` | End-to-end workflow for implementing an `agent:ready` issue |
| `.github/skills/pr-review-checklist-SKILL.md` | Structured checklist for reviewing pull requests |
| `.github/skills/pipeline-debugging-SKILL.md` | Diagnose stalled pipelines and watchdog alerts |

### Policy and Documentation

| File | Purpose |
|------|---------|
| `.github/automation-policy.yml` | Machine-readable boundary between autonomous and human-required actions.  Defines allowed file paths, guardrails, and override commands. |
| `.github/pull_request_template.md` | Standardized PR format with sections for summary, related issue, changes checklist, KPI verification table, and testing checklist. |
| `AGENTS.md` | Cross-tool agent instructions (works with Copilot, Codex, Cursor, and other AI coding tools).  Defines the 8-step workflow, git conventions, label meanings, effort sizing, and project context file locations. |
| `README.md` | This file. |

---

## Labels

The pipeline uses a structured labeling system to coordinate agents:

| Label | Purpose |
|-------|---------|
| `agent:ready` | Issue has no unresolved dependencies and is ready for the coding agent to pick up. |
| `agent:deferred` | Issue is waiting on one or more dependencies to close before it can be worked on. |
| `agent:in-progress` | Issue is currently being worked on by the coding agent. |
| `pipeline` | Issue belongs to the autonomous pipeline (required for auto-dispatch to trigger). |
| `ci-repair` | Issue or PR is in the self-healing CI repair loop. |
| `needs-human` | Escalated issue or PR that requires manual intervention. |
| `review:approved` | PR has been approved by the review agent. |
| `review:changes-requested` | PR has been sent back for changes by the review agent. |
| `role:swe` | Issue originated from a software engineering analysis. |
| `role:business` | Issue originated from a business analysis. |
| `role:datasci` | Issue originated from a data science analysis. |
| `risk:low` / `risk:medium` / `risk:high` | Risk classification of the task. |

---

## Automation Policy

The automation policy (`.github/automation-policy.yml`) defines a strict
boundary between what agents are allowed to do autonomously and what requires
human approval.  The policy defaults to `fail_closed: true` -- any action not
explicitly listed is treated as `human_required`.

### Autonomous Actions

Agents are permitted to perform the following actions without human approval:

- Create feature branches from issues
- Open draft pull requests linking to issues
- Push commits to feature branches (restricted to source, test, and
  documentation paths -- excludes `.github/workflows/**`, `infra/**`, and
  automation policy files)
- Add and remove labels on issues and pull requests
- Close issues when linked pull requests merge
- Assign the coding agent to `agent:ready` issues
- Unlock dependent issues by adding `agent:ready` when all dependencies resolve
- Dispatch the coding agent via the repo-assist workflow
- Undraft pull requests when the agent finishes
- Execute CI checks
- Retry CI failures (up to the configured maximum)
- Post repair commands on issues
- Submit formal GitHub reviews (converting `[FDE-VERDICT]` comments)
- Queue auto-merge on approved pull requests
- Update the pipeline status dashboard issue
- Detect stalled work and re-dispatch

### Human-Required Actions

The following actions are never performed autonomously:

- Modify workflow files (`.github/workflows/**`, `.github/actions/**`)
- Modify the automation policy itself
- Modify infrastructure files (`infra/**`, `pulumi/**`, `terraform/**`,
  `bicep/**`, Dockerfiles, docker-compose files)
- Modify secrets or environment configuration (`.env*`, `**/secrets/**`)
- Modify dependency manifests (`pyproject.toml`, `package.json`,
  `pnpm-lock.yaml`, `uv.lock`, `requirements*.txt`)
- Delete repositories
- Modify branch protection rules
- Change repository visibility
- Force push
- Merge without CI passing
- Create releases

Sensitive file changes detected in a PR are blocked and labeled
`needs-human`.  A repository admin or CODEOWNERS member can override with the
`/approve-sensitive` command.

### Guardrails

| Parameter | Value | Description |
|-----------|-------|-------------|
| `max_repair_attempts` | 2 | Maximum CI repair cycles before escalation |
| `max_watchdog_retries` | 2 | Maximum watchdog re-dispatches for stalled issues |
| `stall_threshold_minutes` | 30 | Minutes of inactivity before an issue is considered stalled |
| `escalation_threshold_minutes` | 120 | Minutes before escalation to `needs-human` |
| `max_concurrent_agents` | 5 | Maximum simultaneous agent dispatches |
| `max_pr_size_lines` | 400 | Maximum lines changed in a single PR |
| `require_ci_pass_for_merge` | true | CI must pass before auto-merge |
| `require_formal_review_for_merge` | true | Formal review required before auto-merge |
| `pipeline_healing_enabled` | true | Enable the self-healing CI repair loop |

---

## Dynamic Files (Pushed Per Project)

After creating a repository from this template, the following files must be
pushed to provide project-specific context to the agents:

| File | Content |
|------|---------|
| `.github/copilot-instructions.md` | Project-specific coding instructions.  Includes technology stack, architectural decisions, naming conventions, and any project-specific patterns the agent must follow. |
| `.github/instructions/system-architecture.instructions.md` | System architecture reference.  Describes modules, data flows, integration points, and the rationale behind design decisions. |
| `.github/specs/swe-analysis.md` | Software engineering analysis or specification.  Contains requirements, constraints, success metrics, and KPIs extracted from the original project brief. |
| `.github/specs/task-plan.md` | Task plan in Markdown format.  Lists every task with its ID, title, description, dependencies, acceptance criteria, file hints, and verification commands. |
| `.github/copilot-setup-steps.yml` | Environment setup workflow for the coding agent.  Installs the correct language runtime, package manager, and project dependencies so the agent can build and test. |

These files are specific to each project and are typically generated by the
`plan2pipeline` library or pushed manually.

---

## Configuration

### Kill Switches

Every workflow supports a repository-level environment variable that
disables it without removing the workflow file:

| Variable | Default | Controls |
|----------|---------|----------|
| `FDE_DISPATCH_ENABLED` | `true` | Auto-dispatch of `agent:ready` issues |
| `FDE_AGENT_ENABLED` | `true` | Coding agent execution (repo-assist) and review agent |
| `FDE_HEALING_ENABLED` | `true` | Self-healing CI repair loop |
| `FDE_WATCHDOG_ENABLED` | `true` | Stall detection and recovery |

Set any variable to `false` in your repository settings to disable the
corresponding workflow.

### Concurrency and Limits

- **MAX_CONCURRENT** (auto-dispatch.yml): Maximum simultaneous agent
  dispatches.  Default: 5.  Overflow issues are labeled `agent:deferred`
  and re-queued when a slot opens.

- **MAX_REPAIR_ATTEMPTS** (ci-repair-router.yml): Maximum CI repair cycles
  per PR.  Default: 2.  After exhausting attempts, the issue is escalated.

- **STALL_THRESHOLD** (pipeline-watchdog.yml): Minutes of inactivity before
  a task is considered stalled.  Default: 30.

---

## Agent Conventions

These conventions are defined in `AGENTS.md` and enforced by the agent prompts
and review agent.

### Branch Naming

All agent branches follow the pattern:

```
issue-<number>-<short-slug>
```

Examples: `issue-1-json-storage`, `issue-5-cli-interface`.

### Commit Format

Conventional commits are required:

```
feat: implement JSON storage module (Fixes #1)
fix: handle empty file on first load
test: add integration tests for CRUD operations
docs: update API reference
chore: configure pre-commit hooks
```

The final commit or PR title must include `Fixes #<number>` to trigger
automatic issue closing.

### PR Requirements

- One issue per pull request.
- Use the pull request template (`.github/pull_request_template.md`).
- Maximum 400 lines changed (larger changes should be split).
- All CI checks must pass.
- KPI verification table must be completed (required by the review agent).

### Verification Commands

The coding agent runs these commands before opening a PR:

```bash
# Python projects
uv run pytest --tb=short -q
uvx pre-commit run --all-files

# TypeScript projects
pnpm test
pnpm lint
pnpm typecheck

# Pre-commit hooks (if configured)
uvx pre-commit install
```

---

## Frequently Asked Questions

**Can I add more agent profiles?**
Yes.  Add new `.agent.md` files to `.github/agents/` and reference them in
your issue dispatch logic.  The template ships with 5 agents (`developer`,
`test-specialist`, `implementation-planner`, `qa-reviewer`, `orchestrator`)
and 4 pipeline skills in `.github/skills/`.  You can add more of either.

**What if my project uses a language other than Python or TypeScript?**
The template is language-agnostic.  Update `.github/copilot-setup-steps.yml`
to install your language runtime and package manager.  Update the
verification commands in `.github/copilot-instructions.md` accordingly.

**How do I handle tasks that are too large?**
The review agent enforces a 400-line PR limit.  If a task produces a PR
exceeding this limit, split the task into smaller sub-tasks with appropriate
dependencies.

**What happens if the coding agent gets stuck in a loop?**
The watchdog (30-min cron) detects stalled issues and re-dispatches them.
After 2 retries, the issue is escalated with `needs-human`.  The CI repair
router similarly limits repair attempts to 2 before escalation.

**Can I use this with tools other than GitHub Copilot?**
The `AGENTS.md` file is designed to work with any AI coding tool that reads
repository context (Copilot, Codex CLI, Cursor, Windsurf, etc.).  The
workflows are currently built around the `gh copilot` CLI but can be adapted.

**How do I disable the pipeline temporarily?**
Set `FDE_DISPATCH_ENABLED=false` in your repository environment variables.
This stops new issues from being dispatched while allowing in-progress work
to complete.

---

## License

This template is provided under the [MIT License](LICENSE).
