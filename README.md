# Sprint Flow

Iterative sprint development workflow plugin for Claude Code. Split PRD into sprints, execute each with a clean-context sub-agent, auto-generate handoffs and completion reports.

## The Problem

When building a project from a PRD, a single AI session runs into context limits. The common workaround — manually copying context prompts into new sessions — is tedious and error-prone.

## The Solution

Sprint Flow automates this pattern:

```
PRD → Split into Sprints → Execute each Sprint via Sub-Agent → Validate → Loop
```

Each sprint gets a sub-agent with clean context and a carefully constructed prompt containing only what it needs.

## Installation

### Step 1: Add marketplace

```
/plugin marketplace add skybao1024/sprint-flow
```

### Step 2: Install plugin

```
/plugin install sprint-flow@sprint-flow-marketplace
```

## Quick Start

```bash
# 1. Initialize: analyze PRD and create sprint plan
/sprint-init docs/my-prd.md

# 2. Review the plan
/sprint-status

# 3. Execute sprints (auto-loops through all)
/sprint-run

# Or execute a specific sprint
/sprint-run 3
```

## Commands

| Command | Description |
|---------|-------------|
| `/sprint-init <prd>` | Analyze PRD, create `.sprint/` with iteration plan and handoffs |
| `/sprint-run [N]` | Execute sprints via sub-agents, validate, loop |
| `/sprint-done [N]` | Manually complete current sprint, generate report |
| `/sprint-status [N]` | Show progress dashboard |

## How It Works

### Architecture

```
Main Agent (Orchestrator)           Sub-Agent (Executor)
┌──────────────────────┐           ┌──────────────────────┐
│ 1. Read iteration    │           │ 1. Read PRD sections │
│    plan              │           │ 2. Read handoff doc  │
│ 2. Read PRD sections │──prompt──▶│ 3. Read CLAUDE.md    │
│ 3. Read design       │           │ 4. Read clarifications│
│    decisions         │           │ 5. Execute tasks     │
│ 4. Run clarification │           │ 6. Update plan       │
│    gate if needed    │           │ 7. Write report      │
│ 5. Read code patterns│           │ 8. Prepare handoff   │
│ 6. Dispatch sub-agent│◀─result──│                      │
│ 7. Validate result   │           └──────────────────────┘
│ 8. Next sprint...    │
└──────────────────────┘
```

### Design Decisions: Think Once, Execute Many

During `/sprint-init`, Sprint Flow performs project-level analysis and generates `.sprint/design-decisions.md`.

This file captures:
- project type and testing standards
- resolved project-level questions
- design decisions with rationale
- sprint contracts between iterations
- clarification escalation policy

Sub-agents consume these decisions during sprint execution so they inherit the global design context without redoing the same analysis every sprint.

### Clarification Gate

Before executing a sprint, `/sprint-run` can run a clarification gate to detect blocking ambiguities for that specific sprint.

The clarification flow works like this:
- identify only **blocking** ambiguities for the current sprint
- write `.sprint/clarification-sprint-{N}.md` when clarification is needed
- ask the user a single batched round of questions
- save answers to `.sprint/sprint-{N}-clarifications.md`
- treat those answers as authoritative during sprint execution

This keeps project-wide reasoning in `design-decisions.md`, sprint-specific decisions in clarification files, and implementation work inside the clean-context executor.

### Key Design: Prompt Quality = Output Quality

Sub-agents don't browse the codebase proactively. The orchestrator must include:
- **Exact file paths** to PRD, CLAUDE.md, pattern files
- **Specific PRD sections** (e.g., "Section 5.3")
- **Complete task lists** pasted into the prompt
- **Concrete constraints** from project conventions

### Generated Files

```
.sprint/
├── config.json                      # Project config and current state
├── design-decisions.md              # Global decisions, contracts, and escalation policy
├── iteration-plan.md                # Master plan with all sprints
├── handoff-sprint-0.md              # Sprint 0 detailed instructions
├── handoff-sprint-1.md              # Sprint 1 detailed instructions
├── clarification-sprint-1.md        # Blocking questions for a sprint when needed
├── sprint-1-clarifications.md       # User answers for sprint-specific blockers
├── sprint-0-completion-report.md    # What Sprint 0 accomplished
├── sprint-1-completion-report.md    # What Sprint 1 accomplished
└── ...
```

### Sprint Splitting Principles

- Sprint 0: critical bugs/security or foundational setup
- Each sprint: 2-8 minutes of AI work
- Clear boundaries (full module or well-defined subset)
- Strictly ordered dependencies
- Each sprint produces testable output
- Each sprint includes unit tests for the code it produces

## Requirements

- Claude Code CLI
- A PRD document in your project

## License

MIT
