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

### From a marketplace

```
/plugin install sprint-flow@<marketplace-name>
```

### From GitHub

```
/plugin install-from-github <your-username>/sprint-flow
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
│ 3. Read CLAUDE.md    │           │ 4. Execute tasks     │
│ 4. Read code patterns│           │ 5. Update plan       │
│ 5. Construct prompt  │           │ 6. Write report      │
│ 6. Dispatch sub-agent│◀─result──│ 7. Prepare handoff   │
│ 7. Validate result   │           └──────────────────────┘
│ 8. Next sprint...    │
└──────────────────────┘
```

### Key Design: Prompt Quality = Output Quality

Sub-agents don't browse the codebase proactively. The orchestrator must include:
- **Exact file paths** to PRD, CLAUDE.md, pattern files
- **Specific PRD sections** (e.g., "Section 5.3")
- **Complete task lists** pasted into the prompt
- **Concrete constraints** from project conventions

### Generated Files

```
.sprint/
├── config.json                    # Project config and current state
├── iteration-plan.md              # Master plan with all sprints
├── handoff-sprint-0.md            # Sprint 0 detailed instructions
├── handoff-sprint-1.md            # Sprint 1 detailed instructions
├── sprint-0-completion-report.md  # What Sprint 0 accomplished
├── sprint-1-completion-report.md  # What Sprint 1 accomplished
└── ...
```

### Sprint Splitting Principles

- Sprint 0: critical bugs/security or foundational setup
- Each sprint: 2-4 hours of AI work
- Clear boundaries (full module or well-defined subset)
- Strictly ordered dependencies
- Each sprint produces testable output

## Requirements

- Claude Code CLI
- A PRD document in your project

## Version

0.1.0 — Initial release

## License

MIT
