---
description: "Complete current sprint, generate reports, and prepare next sprint handoff"
argument-hint: "[sprint-number]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash]
---

# /sprint-done — Complete Current Sprint

Finalize the current sprint and prepare handoff for the next one.

**Arguments**: $ARGUMENTS (optional: sprint number, defaults to current from config)

## Step 1: Determine Current Sprint

1. Read `.sprint/config.json` for `current_sprint`
2. If argument provided, use that sprint number
3. Read `.sprint/iteration-plan.md` for sprint details

## Step 2: Gather Completion Data

1. Read `.sprint/handoff-sprint-{N}.md` for what was planned
2. Read `.sprint/design-decisions.md` — check sprint contracts for this sprint (what was expected to be produced)
3. Run `git diff --stat` or `git log --oneline -20` to see recent changes (if git repo)
4. Scan for files created/modified during this sprint
5. Check which acceptance criteria are met
6. Verify sprint contract compliance — did this sprint produce the deliverables that downstream sprints expect?

## Step 3: Generate Completion Report

Create `.sprint/sprint-{N}-completion-report.md`:

```markdown
# Sprint {N} Completion Report: {Sprint Name}

**Completed**: {date}

## Summary
{1-2 sentence summary}

## Completed Tasks
| ID | Task | Status | Notes |
|----|------|--------|-------|
| S{N}-T1 | {task} | Completed | {notes} |

## Files Changed
### New Files
- `path/to/file` — {description}
### Modified Files
- `path/to/file` — {what changed}

## Acceptance Criteria
- [x] {met criterion}
- [ ] {unmet — explain why}

## Sprint Contract Compliance
- **Expected output**: {from design-decisions.md sprint contracts}
- **Actual output**: {what was actually produced}
- **Status**: Met / Partially Met / Deviated
- **Deviations**: {if any, explain what changed and why}

## Deviations from Plan
{differences from handoff}

## Issues Encountered
{problems and resolutions}

## Impact on Next Sprint
{considerations for next sprint, including any sprint contract deviations that affect downstream work}
```

## Step 4: Update Iteration Plan

Edit `.sprint/iteration-plan.md`:
- Change Sprint {N} status to "Completed"
- Add completion date

## Step 5: Prepare Next Sprint Handoff

If more sprints remain:
1. Read PRD sections for Sprint {N+1}
2. Create/update `.sprint/handoff-sprint-{N+1}.md`

## Step 6: Update Config

Update `.sprint/config.json`:
- Always set `current_sprint` to N+1 (even if N+1 equals total_sprints)
- If `current_sprint` >= `total_sprints`, set `status` to "completed"
- Example: for 8 sprints (0-7), after Sprint 7 completes → `current_sprint: 8`, `status: "completed"`

## Step 7: Output Next Session Prompt

```
--- NEXT SESSION PROMPT (copy to new session) ---

Continuing development on {project_name}. Sprint {N} ({name}) completed.

To start Sprint {N+1}:
1. Read `.sprint/config.json`
2. Read `.sprint/iteration-plan.md`
3. Read `.sprint/sprint-{N}-completion-report.md`
4. Read `.sprint/handoff-sprint-{N+1}.md`
5. Read PRD at `{prd_path}`, focus on: {sections}
6. Read the project instruction file at `{instruction_file_path}` if one is configured

Then execute the tasks in the handoff document.

--- END ---
```

Or if all sprints complete:
```
All {total} sprints completed! Review .sprint/ for full record.
```
