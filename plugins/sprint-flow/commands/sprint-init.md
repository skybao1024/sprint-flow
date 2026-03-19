---
description: "Initialize iterative sprint workflow from a PRD document"
argument-hint: <prd-file-path>
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent, AskUserQuestion]
---

# /sprint-init — Initialize Sprint Workflow

You are setting up an iterative sprint development workflow for this project. This workflow enables AI-driven development where work is split into sprints, each executed by a sub-agent with clean context.

## Your Task

### 1. Locate the PRD document

The user may provide a PRD path as argument: `$ARGUMENTS`

If no argument is provided, search for PRD files:
- Look for files matching: `*prd*`, `*PRD*`, `*requirement*`, `*spec*` in `docs/`, `doc/`, root directory
- If multiple found, ask the user which one to use
- If none found, ask the user for the path

### 2. Read and analyze the PRD

Read the entire PRD document. Identify:
- Project name and description
- Major modules/features (these become sprint candidates)
- Dependencies between modules
- Tech stack and constraints
- Any existing CLAUDE.md conventions

### 3. Create the `.sprint/` directory structure

Create the following files in the project root:

**`.sprint/config.json`**:
```json
{
  "project_name": "<from PRD>",
  "prd_path": "<relative path to PRD>",
  "tech_stack": "<detected tech stack>",
  "total_sprints": <number>,
  "current_sprint": 0,
  "status": "initialized",
  "created_at": "<ISO date>",
  "conventions": {
    "claude_md_path": "<path to CLAUDE.md if exists>",
    "test_command": "<detected test command>",
    "build_command": "<detected build command>",
    "key_patterns": ["<important patterns from CLAUDE.md or codebase>"]
  }
}
```

**`.sprint/iteration-plan.md`** — structured iteration plan:

```markdown
# <Project Name> — Iteration Plan

**PRD Source**: <prd_path>
**Created**: <date>
**Status**: Sprint 0 pending

## Sprint Overview

| Sprint | Name | Objective | Priority | Dependencies | Status |
|--------|------|-----------|----------|-------------|--------|
| 0 | <name> | <objective> | CRITICAL/HIGH/MEDIUM | None | Pending |
| 1 | <name> | <objective> | ... | Sprint 0 | Pending |

## Sprint Details

### Sprint 0: <Name>

**Objective**: <one-line objective>
**Priority**: <CRITICAL/HIGH/MEDIUM>
**Dependencies**: None
**PRD Sections**: <section references>

#### Task List

| ID | Task | Target Files | PRD Reference | Priority |
|----|------|-------------|---------------|----------|
| S0-T1 | <task> | <files> | Section X.Y | HIGH |

#### Acceptance Criteria
- [ ] <criterion 1>
- [ ] <criterion 2>
```

**`.sprint/handoff-sprint-0.md`** — first sprint handoff:

```markdown
# Sprint 0 Handoff: <Sprint Name>

**Created**: <date>
**Status**: Ready to start
**Objective**: <objective>

## Required Reading

Before starting, read these documents:
1. **PRD**: `<prd_path>` — Focus on: <specific sections>
2. **CLAUDE.md**: `<path>` — Coding standards
3. **Iteration Plan**: `.sprint/iteration-plan.md`

## Task List

| ID | Task | Target Files | Detailed Instructions | Priority |
|----|------|-------------|----------------------|----------|
| S0-T1 | ... | ... | ... | ... |

## Implementation Guide

### Existing Patterns to Follow
- <pattern reference from codebase>

### Key Constraints
- <constraint from PRD>
- <constraint from CLAUDE.md>

## Acceptance Criteria
- [ ] <criteria>

## After Completion
1. Update `.sprint/iteration-plan.md` — Mark Sprint 0 as completed
2. Create `.sprint/sprint-0-completion-report.md`
3. Create `.sprint/handoff-sprint-1.md` if more sprints remain
```

### 4. Sprint splitting principles

Follow these rules when splitting PRD into sprints:
- **Sprint 0** should always be critical bugs/security fixes if any exist, or foundational setup
- Each sprint should be completable in **2-4 hours of AI work**
- Sprints must have **clear boundaries** — a sprint either fully implements a module or a well-defined subset
- Dependencies must be **strictly ordered** — no circular dependencies
- Each sprint should produce **testable, runnable output**
- **Each sprint MUST include unit tests for the code it produces.** Tests are part of the sprint deliverables, not a separate sprint. Include "unit tests pass" as an acceptance criterion for every sprint.
- Aim for **5-8 sprints** for a typical MVP, adjust based on complexity

### 5. Output summary

After creating all files, output:
- Total sprints planned
- Brief description of each sprint
- Command to start: "Run `/sprint-run` to begin executing sprints, or `/sprint-status` to review the plan"

## Important

- Do NOT start executing any sprint tasks. This command only sets up the plan.
- If the project already has `.sprint/config.json`, warn the user and ask if they want to reinitialize.
- Make the iteration plan detailed enough that a sub-agent with NO prior context can execute each sprint independently.
