# plan2pipeline

> Bootstrapped by [FDE Copilot](https://github.com/microsoft/fde-listen2spec) — autonomous spec-to-code pipeline.

## Pipeline Status

This repository uses an autonomous agent pipeline:

1. **Issues** — each issue is a self-contained agent prompt with acceptance criteria
2. **Developer agent** — picks `agent:ready` issues and opens PRs
3. **QA reviewer agent** — reviews PRs for correctness and coverage
4. **Auto-merge** — approved PRs are merged automatically

### Labels

| Label | Meaning |
|-------|---------|
| `agent:ready` | Issue is ready for the coding agent to pick up |
| `agent:deferred` | Issue is waiting on dependencies |
| `pipeline` | Issue belongs to the autonomous pipeline |

## Getting Started

See [AGENTS.md](AGENTS.md) for the full agent workflow and conventions.

## Project Context

- `.github/copilot-instructions.md` — project-specific coding instructions
- `.github/instructions/system-architecture.instructions.md` — architecture reference
- `.github/specs/` — original FDE spec analyses
