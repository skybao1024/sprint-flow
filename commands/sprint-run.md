---
description: "Execute sprints iteratively using sub-agents with clean context"
argument-hint: "[sprint-number]"
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /sprint-run — Sprint Execution Orchestrator

You are the **orchestrator** of an iterative sprint development workflow. Your job is to dispatch sub-agents to execute each sprint, validate their work, and coordinate overall progress.

**Arguments**: $ARGUMENTS (optional: sprint number to start from, e.g., `/sprint-run 3`)

## Phase 1: Load Project Context

1. Read `.sprint/config.json` — if missing, tell user to run `/sprint-init` first
2. Read `.sprint/iteration-plan.md` for the full sprint plan
3. Determine which sprint to execute next:
   - If argument provided, start from that sprint number
   - Otherwise, find the first sprint with status "Pending"
4. If all sprints are completed, inform the user and exit

## Phase 2: Sprint Execution Loop

For each pending sprint, execute this cycle:

### Step 2.1: Prepare Sub-Agent Context

Before dispatching, gather ALL necessary information:

1. Read `.sprint/handoff-sprint-{N}.md` for detailed instructions
2. Read the PRD file (path from config.json) — identify sections relevant to this sprint
3. Read `CLAUDE.md` (if exists) for project conventions
4. If N > 0, read `.sprint/sprint-{N-1}-completion-report.md` for prior context
5. Scan the codebase for patterns:
   - Use Glob/Grep to find example files referenced in the handoff
   - Identify 1-2 representative files as pattern references

### Step 2.2: Construct Sub-Agent Prompt

**THIS IS THE MOST CRITICAL STEP.** Build the prompt using this template:

```
You are a developer working on the {project_name} project.
Your task is to complete Sprint {N}: {sprint_name}.

## Project Context

**Project**: {project_name}
**Tech Stack**: {tech_stack}
**Your Objective**: {sprint_objective}

## MANDATORY: Read These Documents First

You MUST read these files IN ORDER before writing any code:

1. `{prd_path}` — The PRD document. Focus on:
   {specific_section_list}

2. `{claude_md_path}` — Project coding standards.

3. `.sprint/handoff-sprint-{N}.md` — Your detailed task list.

{if N > 0}
4. `.sprint/sprint-{N-1}-completion-report.md` — Previous sprint results.
{endif}

## Pattern References

Study these files to understand coding patterns:
{pattern_file_list_with_explanations}

## Your Task List

{paste full task list from handoff}

## Key Constraints

{constraints_from_prd_and_claude_md}

## Execution Rules

1. Read ALL listed documents before writing any code.
2. Match the style of pattern reference files exactly.
3. Complete each task fully before moving to the next.
4. After ALL tasks done, update `.sprint/iteration-plan.md`:
   - Change Sprint {N} status to "Completed", add date
5. Create `.sprint/sprint-{N}-completion-report.md` with:
   - All changes made (files created, modified, deleted)
   - Any deviations from plan and why
   - Issues encountered and resolutions
   - Risks or concerns for next sprints
6. If `.sprint/handoff-sprint-{N+1}.md` exists, review and update it.
   If it doesn't exist and more sprints remain, create it.

## Do NOT
- Guess requirements — check the PRD
- Skip reading documents
- Modify files outside sprint scope unless necessary
- Leave TODO comments — implement fully
```

### Step 2.3: Dispatch Sub-Agent

```
Agent(
  prompt: <constructed prompt>,
  description: "Execute Sprint {N}: {sprint_name}",
  model: "sonnet"
)
```

Use `model: "sonnet"` by default. Use "opus" only for complex architectural sprints or after a sonnet attempt fails.

### Step 2.4: Validate Sprint Completion

After sub-agent returns:

1. Read `.sprint/iteration-plan.md` — verify Sprint N marked completed
2. Read `.sprint/sprint-{N}-completion-report.md` — verify it exists
3. Check handoff for Sprint N+1 exists (if more remain)
4. Spot-check: Glob for files in completion report, read 1-2 key files

### Step 2.5: Handle Failures

- **Minor issues**: Fix directly (edit files, update docs)
- **Major issues**: Re-dispatch sub-agent with more specific prompt
- **Blockers**: Inform user and ask for guidance

### Step 2.6: Report and Continue

Output brief summary:
```
Sprint {N} ({sprint_name}): Completed
- Files changed: X
- Key deliverables: ...
- Issues: none / <description>
```

Then ask: "Continue with Sprint {N+1}? (y/n)"
- Yes → loop to Step 2.1 for Sprint N+1
- No → output current status and exit

## Phase 3: All Sprints Complete

```
All {total} sprints completed!

Sprint 0: {name} — Completed
Sprint 1: {name} — Completed
...
```

Update `.sprint/config.json`: set `status` to "completed"

## Key Principles

- Each sub-agent gets clean context — this is by design
- The prompt you construct IS the sub-agent's entire knowledge
- If a sprint fails, make the prompt more specific (actual paths, actual sections)
- Never be vague in sub-agent prompts — include concrete file paths and constraints
