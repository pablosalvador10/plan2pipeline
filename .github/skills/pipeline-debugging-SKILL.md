---
name: pipeline-debugging
description: >-
  Diagnose stalled or broken autonomous pipelines. Use when the pipeline stops
  making progress, issues are stuck, the watchdog fires, or agents appear
  idle. Triggers: pipeline stalled, watchdog alert, no progress, stuck issue,
  agent idle, pipeline broken.
argument-hint: Describe the symptom or provide the stalled issue number
---

## Purpose

Systematic approach to diagnosing why the autonomous pipeline has stopped
making progress and how to recover.

## When to Use

- The pipeline-watchdog fires a stall alert
- No PRs have been opened for 30+ minutes despite `agent:ready` issues
- An issue has been `agent:in-progress` for over 30 minutes
- The pipeline-status dashboard shows zero movement

## Flow

1. **Check the Pipeline Status issue** — look for the `[FDE] Pipeline Status`
   issue, note completion counts and any flagged stalls
2. **Identify the blocked issue(s):**
   ```bash
   gh issue list --label "agent:in-progress" --state open --json number,title,updatedAt
   gh issue list --label "agent:ready" --state open --json number,title
   ```
3. **Check for dependency deadlocks:**
   - Look for circular dependencies (A depends on B, B depends on A)
   - Look for unresolved dependencies on closed issues that weren't detected
4. **Check workflow run history:**
   ```bash
   gh run list --workflow=auto-dispatch.yml --limit=5
   gh run list --workflow=repo-assist.yml --limit=5
   ```
5. **Check for concurrency saturation:**
   - Count active `agent:in-progress` issues — if at MAX_CONCURRENT (5),
     the queue is full and new dispatches are deferred
6. **Common fixes:**
   - **Stuck dispatch:** Re-label the issue: remove `agent:in-progress`,
     add `agent:ready`
   - **Failed workflow:** Re-run the failed workflow from the Actions tab
   - **Dependency not detected:** Manually close the blocking issue or
     relabel the dependent issue as `agent:ready`
   - **Concurrency full:** Wait for an in-progress issue to complete, or
     increase `MAX_CONCURRENT` in `auto-dispatch.yml`

## Guardrails

- **Max 2 watchdog retries** per issue before escalating to `needs-human`
- **Never force-close issues** to unblock — investigate the root cause
- **Check workflow permissions** if dispatch silently fails
