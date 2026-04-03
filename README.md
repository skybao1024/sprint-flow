# Sprint Flow

Iterative sprint development workflow for Claude Code, with a Codex-compatible integration path. Split PRD into sprints, execute each with a clean-context delegated executor when the host supports it, and auto-generate handoffs and completion reports.

## The Problem

When building a project from a PRD, a single AI session runs into context limits. The common workaround — manually copying context prompts into new sessions — is tedious and error-prone.

## The Solution

Sprint Flow automates this pattern:

```
PRD → Split into Sprints → Execute each Sprint via Delegated Executor → Validate → Loop
```

Each sprint gets a delegated executor with clean context when the host supports it, plus a carefully constructed prompt containing only what it needs. On hosts without equivalent delegation, the same sprint contract can still run in the main session.

## Installation

### Claude Code

#### Step 1: Add marketplace

```
/plugin marketplace add skybao1024/sprint-flow
```

#### Step 2: Install plugin

```
/plugin install sprint-flow@sprint-flow-marketplace
```

### Codex

Sprint Flow ships a Codex Skills entrypoint at `.codex/skills/sprint-flow`.

For end users, Codex support is distributed as a **Skill**, separate from the Claude Code plugin wrapper:
- skill entrypoint: `.codex/skills/sprint-flow/SKILL.md`
- command wrappers: `.codex/skills/sprint-flow/commands/*.md`

Install the repository's `sprint-flow` skill into a Codex skills discovery path supported by your Codex installation, then restart Codex so it re-discovers the skill metadata.

See `.codex/INSTALL.md` for the supported layout, what gets installed, and local setup examples.

Codex support reuses the same `.sprint/` workflow artifacts, but installation and capability mapping stay separate from the Claude Code plugin wrapper.

## Quick Start

Claude Code commands:

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
Main Agent (Orchestrator)           Delegated Executor
┌──────────────────────┐           ┌──────────────────────┐
│ 1. Read iteration    │           │ 1. Read PRD sections │
│    plan              │           │ 2. Read handoff doc  │
│ 2. Read PRD sections │──prompt──▶│ 3. Read instruction  │
│ 3. Read design       │           │    file if present   │
│    decisions         │           │ 4. Read clarifications│
│ 4. Run clarification │           │ 5. Execute tasks     │
│    gate if needed    │           │ 6. Update plan       │
│ 5. Read code patterns│           │ 7. Write report      │
│ 6. Dispatch executor │◀─result──│ 8. Prepare handoff   │
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

Delegated executors consume these decisions during sprint execution so they inherit the global design context without redoing the same analysis every sprint.

### Clarification Gate

Before executing a sprint, `/sprint-run` can run a clarification gate to detect blocking ambiguities for that specific sprint.

The clarification flow works like this:
- identify only **blocking** ambiguities for the current sprint
- write `.sprint/clarification-sprint-{N}.md` when clarification is needed
- ask the user a single batched round of questions
- save answers to `.sprint/sprint-{N}-clarifications.md`
- treat those answers as authoritative during sprint execution

This keeps project-wide reasoning in `design-decisions.md`, sprint-specific decisions in clarification files, and implementation work inside the delegated executor.

### Key Design: Prompt Quality = Output Quality

Delegated executors don't browse the codebase proactively. The orchestrator must include:
- **Exact file paths** to PRD, instruction file, pattern files
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

- Claude Code CLI for the current plugin wrapper
- Codex, if you use the Codex wrapper described in `.codex/INSTALL.md`
- A PRD document in your project

## License

MIT
