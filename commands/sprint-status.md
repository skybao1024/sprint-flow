---
description: "Show current sprint workflow status and progress overview"
argument-hint: "[sprint-number]"
allowed-tools: [Read, Glob]
---

# /sprint-status — Sprint Workflow Status

Show the current state of the iterative sprint workflow.

## Steps

1. Read `.sprint/config.json`
   - If missing: "No sprint workflow found. Run `/sprint-init <prd-path>` to set up."

2. Read `.sprint/iteration-plan.md`

3. Gather status:
   - Count total, completed, pending sprints
   - Identify current sprint (first pending)
   - Check completion reports and handoffs exist

4. Output:

```
Sprint Workflow: {project_name}
PRD: {prd_path}

Progress: [{completed}/{total}] {'='*completed}{'.'*remaining} {pct}%

| Sprint | Name               | Status    | Completed  |
|--------|--------------------|-----------|------------|
| 0      | {name}             | Completed | 2026-03-18 |
| 1      | {name}             | Completed | 2026-03-18 |
| 2      | {name}             | Pending   |            |

Current: Sprint {N} — {name}
Handoff: {Yes/No}

Commands:
  /sprint-run       Execute remaining sprints with sub-agents
  /sprint-run {N}   Execute specific sprint
  /sprint-done      Manually complete current sprint
```

## If $ARGUMENTS contains a sprint number

Show detailed view: read handoff + completion report, show task-level progress.
