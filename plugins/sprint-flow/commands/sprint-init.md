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
- Any existing project instruction file conventions. Host selection is explicit: use `CLAUDE.md` for Claude Code and `AGENTS.md` for Codex.

### 3. Design Analysis (Think Once, Execute Many)

Before splitting into sprints, perform deep analysis to resolve **project-level ambiguities** and establish design context. This analysis is done **once** and produces `.sprint/design-decisions.md`, which all sub-agents will consume.

**Important boundary**:

- `/sprint-init` resolves **project-level ambiguity** — product scope, tech stack, architecture direction, testing standards, deployment assumptions, and sprint contract design.
- `/sprint-run` resolves **sprint-level ambiguity** through a clarification gate before execution when a specific sprint still has blocking feature-level questions.
- Do NOT try to fully resolve every feature detail during init. The goal is to make sprint execution well-scoped, not to replace per-sprint detailed design review.

#### 3a. Host Instruction File Selection

Before generating `.sprint/config.json`, determine the active host and set `runtime.instruction_file_path` using these rules:

- If the host is **Claude Code**, set `runtime.instruction_file_path` to `CLAUDE.md` if that file exists.
- If the host is **Codex**, set `runtime.instruction_file_path` to `AGENTS.md` if that file exists.
- If the expected host-specific file does not exist, leave `runtime.instruction_file_path` empty.
- Do NOT substitute another host's instruction file automatically.

This keeps one canonical field for downstream prompts while preserving host-specific rule discovery.

#### 3b. Project Type Detection & Development Standards

Analyze the PRD to detect the project type and record it in `config.json` as `project_type`. Apply **type-specific development standards** based on the detected type:

| Project Type | Testing Requirements |
|---|---|
| **Backend** | Mandatory unit tests for all business logic, input validation tests, error handling tests. Must cover: happy path, edge cases, error cases. Test coverage is a hard acceptance criterion. |
| **Frontend** | Component rendering tests, user interaction tests. Visual/layout testing encouraged but not mandatory. |
| **CLI** | Command parsing tests, output format tests, error message tests. |
| **Data Pipeline** | Data transformation tests, schema validation tests, edge case data tests. |
| **Full-stack** | Backend standards for API layer + frontend standards for UI layer. |
| **Other** | Determine appropriate testing standards based on project nature and document them. |

If the project type is ambiguous, ask the user to confirm.

#### 3c. Ambiguity Detection

Identify unclear, missing, or conflicting points in the PRD. Categorize into:

- **Tech decisions**: Unresolved technology choices
- **Business logic gaps**: Missing rules, edge cases, or workflows
- **Scope boundaries**: Unclear what's in/out of scope

#### 3d. Layered Questioning

Use the host's interactive question capability (for Claude Code, `AskUserQuestion`) to resolve ambiguities in **batched rounds** (not one-at-a-time):

- **Round 1**: Independent questions (2-4 per batch, multiple choice preferred) — tech stack, deployment targets, scope confirmations
- **Round 2** (if needed): Follow-up questions that depend on Round 1 answers — e.g., if user chose JWT, ask about token refresh strategy
- **Round 3** (if needed): Final clarifications
- **Maximum 3 rounds total** to keep init phase efficient

#### 3e. Design Decisions, Sprint Contracts & Escalation Policy

Based on PRD analysis and user answers:

1. Record all design decisions with rationale
2. Define **sprint contracts** — what each sprint produces that subsequent sprints consume. These are NOT limited to APIs; they describe whatever deliverables connect sprints (components, modules, data structures, config files, schemas, CLI interfaces, etc.)
3. Define a **clarification escalation policy** for sprint execution:
   - Escalate only when ambiguity affects business rules, MVP scope boundaries, schema/data invariants, external API contracts, auth/security behavior, or downstream sprint contracts
   - For non-blocking ambiguities, the sprint executor should choose the most conservative implementation and document assumptions in the completion report

#### 3f. Generate `.sprint/design-decisions.md`

```markdown
# Design Decisions — <Project Name>

**Project Type**: <detected type>
**Created**: <date>

## Development Standards

{type-specific testing and development requirements from Step 3a}

## Resolved Questions

| Question | Answer | Rationale | Decided |
|----------|--------|-----------|---------|
| <question> | <answer> | <why> | <date> |

## Design Decisions

### <Decision Title>
- **Decision**: <what was decided>
- **Rationale**: <why>
- **Alternatives considered**: <what else was possible>

## Sprint Contracts

### Sprint 0 → Sprint 1
- **Sprint 0 produces**: <deliverables — adapt to project type>
- **Sprint 1 expects**: <what it will consume>

### Sprint 1 → Sprint 2
...

## Clarification Escalation Policy

Escalate to the user during sprint execution only when ambiguity affects:
- business rules or workflow logic
- MVP scope boundaries
- schema or data model invariants
- external API contracts
- auth / permission / security behavior
- downstream sprint contracts

For non-blocking ambiguities:
- choose the most conservative implementation
- document assumptions in the sprint completion report
- avoid interrupting execution

## Key Constraints

- <constraint from PRD>
- <constraint from user answers>
- <constraint from tech stack>
```

### 4. Create the `.sprint/` directory structure

Create the following files in the project root:

**`.sprint/config.json`**:
```json
{
  "project_name": "<from PRD>",
  "prd_path": "<relative path to PRD>",
  "tech_stack": "<detected tech stack>",
  "project_type": "<backend|frontend|cli|data-pipeline|mobile|full-stack|other>",
  "total_sprints": <number>,
  "current_sprint": 0,
  "status": "initialized",
  "created_at": "<ISO date>",
  "runtime": {
    "host": "<claude-code|codex|other>",
    "instruction_file_path": "<host-specific path: Claude Code uses CLAUDE.md, Codex uses AGENTS.md>",
    "capabilities": {
      "delegation": true,
      "interactive_questions": true
    }
  },
  "conventions": {
    "test_command": "<detected test command>",
    "build_command": "<detected build command>",
    "key_patterns": ["<important patterns from the selected host instruction file or codebase>"]
  }
}
```

The selected `runtime.instruction_file_path` must always match the active host:

- Claude Code → `CLAUDE.md`
- Codex → `AGENTS.md`

Do not write another host's instruction file into this field.

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
2. **Instruction File**: `<runtime.instruction_file_path>` — Current host's project-specific coding standards and conventions
3. **Iteration Plan**: `.sprint/iteration-plan.md`

## Task List

| ID | Task | Target Files | Detailed Instructions | Priority |
|----|------|-------------|----------------------|----------|
| S0-T1 | ... | ... | ... | ... |

## Design Context

{Relevant decisions from `.sprint/design-decisions.md` for this sprint — include design decisions, rationale, and any resolved questions that affect this sprint's implementation.}

## Sprint Contracts

- **Input**: {What this sprint receives from previous sprints — could be modules, schemas, components, config, etc.}
- **Output**: {What this sprint must produce for downstream sprints to consume}

## Implementation Guide

### Existing Patterns to Follow
- <pattern reference from codebase>

### Key Constraints
- <constraint from PRD>
- <constraint from the project instruction file>

## Test Scenarios

{Explicit list of test scenarios this sprint MUST cover, generated per the development standards for this project type. Do NOT use vague "write unit tests" — list specific scenarios.}

| ID | Scenario | Type | Expected Behavior |
|----|----------|------|-------------------|
| S0-TS1 | <scenario> | happy path / edge case / error | <expected> |

## Acceptance Criteria
- [ ] <criteria>
- [ ] All test scenarios above have passing tests

## After Completion
1. Update `.sprint/iteration-plan.md` — Mark Sprint 0 as completed
2. Create `.sprint/sprint-0-completion-report.md`
3. Create `.sprint/handoff-sprint-1.md` if more sprints remain
```

### 5. Sprint splitting principles

Follow these rules when splitting PRD into sprints:

- **Sprint 0** should always be critical bugs/security fixes if any exist, or foundational setup
- Each sprint should be completable in **2-4 hours of AI work**
- Sprints must have **clear boundaries** — a sprint either fully implements a module or a well-defined subset
- Dependencies must be **strictly ordered** — no circular dependencies
- Each sprint should produce **testable, runnable output**
- **Each sprint MUST include tests per the development standards** defined in `.sprint/design-decisions.md`. Tests are part of the sprint deliverables, not a separate sprint. The handoff's `## Test Scenarios` table lists specific scenarios that must be covered.
- Aim for **5-8 sprints** for a typical MVP, adjust based on complexity

### 6. Output summary

After creating all files, output:

- Total sprints planned
- Brief description of each sprint
- Note that project-level ambiguities were resolved here, while sprint-level blockers will be handled by `/sprint-flow:sprint-run` through the clarification gate if needed
- Command to start: "Run `/sprint-flow:sprint-run` to begin executing sprints, or `/sprint-flow:sprint-status` to review the plan"
- If the current host uses different command discovery or install steps, follow that host's wrapper documentation while reusing the same `.sprint/` workflow artifacts

## Important

- Do NOT start executing any sprint tasks. This command only sets up the plan.
- If the project already has `.sprint/config.json`, do NOT blindly reinitialize. First read the existing `.sprint/config.json` and `.sprint/iteration-plan.md` and branch as follows:
  - If the existing sprint workflow is **completed** (`status: completed` or all planned sprints are marked completed), tell the user an earlier sprint plan already finished and ask whether to **clean up the old `.sprint/` documents and create a new plan**.
    - Preferred wording: "An earlier sprint plan is already completed. Should I clean the old `.sprint/` documents and initialize a new plan?"
  - If the existing sprint workflow is **not completed** (`status` is not `completed` or there are still Pending / In Progress sprints), tell the user there is an unfinished plan and ask whether to **continue the existing plan first** or **clean up the old `.sprint/` documents and create a new one**.
    - Preferred wording: "There is an unfinished sprint plan. Do you want to continue it first, or clean the old `.sprint/` documents and start a new plan?"
  - If the user chooses cleanup, remove only sprint workflow artifacts under `.sprint/` before reinitializing. Do NOT touch project source code.
  - If the user does not confirm cleanup, do NOT overwrite the existing sprint documents.
- Make the iteration plan detailed enough that a sub-agent with NO prior context can execute each sprint independently.
