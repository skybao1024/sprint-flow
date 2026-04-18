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
| `/sprint-init <prd>` | Analyze PRD, classify sprint types, create `.sprint/` with iteration plan and handoffs |
| `/sprint-run [N]` | Execute sprints via sub-agents with type-specific context, validate, loop |
| `/sprint-done [N]` | Manually complete current sprint, generate report |
| `/sprint-status [N]` | Show progress dashboard |

## Features

### Persona-Based Professional Workflow

Sprint Flow uses a **persona system** to provide specialized expertise based on sprint type. Each persona has:
- Professional identity and responsibilities
- Domain-specific knowledge and best practices
- Specialized workflow and quality standards
- Validation checklists tailored to their domain

**Available Personas**:
- **Frontend Developer**: UI implementation, design system compliance, accessibility
- **Backend Developer**: API design, business logic, security, database
- **Fullstack Developer**: End-to-end feature implementation
- **Design Specialist** (Assistant): Design system guidance and validation
- **API Integration Specialist** (Assistant): API contract compliance and validation

**Persona Activation**:
- `frontend` sprint → `frontend-developer` + `design-specialist` + `api-integration-specialist`
- `backend` sprint → `backend-developer`
- `fullstack` sprint → `fullstack-developer` + `design-specialist` + `api-integration-specialist`

### Sprint Type Classification

Sprint Flow automatically classifies each sprint as `frontend`, `backend`, or `fullstack` based on:
- Task descriptions and keywords
- Target file patterns (`.jsx`, `.tsx`, `.go`, `.py`, etc.)
- PRD content analysis

Different sprint types activate different personas with specialized expertise.

### Frontend Development Enhancements

For frontend sprints, the **frontend-developer** persona is supported by two specialist advisors:

#### Design Specialist Support
- Auto-detects design systems (Tailwind, Material-UI, Ant Design, etc.)
- Provides design system guidance and best practices
- Validates design system compliance
- Ensures WCAG 2.1 AA accessibility compliance
- Reviews responsive design implementation

#### API Integration Specialist Support
- Auto-discovers Swagger/OpenAPI documentation
- Parses API contracts for relevant endpoints
- Validates API endpoint usage (prevents invented APIs)
- Ensures request/response schema compliance
- Guides proper authentication implementation

#### Frontend-Specific Validation
Frontend sprint completion reports include:
- Design system pattern compliance
- API contract adherence (no invented endpoints)
- Accessibility requirements (WCAG 2.1 AA)
- Responsive design implementation
- Design reference consultation

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
├── config.json                      # Project config, sprint type flags, and frontend context
├── design-decisions.md              # Global decisions, contracts, and frontend context (if applicable)
├── iteration-plan.md                # Master plan with all sprints and their types
├── handoff-sprint-0.md              # Sprint 0 detailed instructions (persona-specific)
├── handoff-sprint-1.md              # Sprint 1 detailed instructions (persona-specific)
├── clarification-sprint-1.md        # Blocking questions for a sprint when needed
├── sprint-1-clarifications.md       # User answers for sprint-specific blockers
├── sprint-0-completion-report.md    # What Sprint 0 accomplished (persona-specific validation)
├── sprint-1-completion-report.md    # What Sprint 1 accomplished (persona-specific validation)
└── ...
```

**Persona Files**:
```
plugins/sprint-flow/personas/
├── frontend-developer.md            # Frontend development persona
├── backend-developer.md             # Backend development persona
├── fullstack-developer.md           # Fullstack development persona
├── design-specialist.md             # Design expert (frontend assistant)
└── api-integration-specialist.md    # API integration expert (frontend assistant)
```

#### config.json Schema

```json
{
  "project_name": "...",
  "project_type": "fullstack",
  "has_frontend_sprints": true,
  "has_backend_sprints": true,
  "frontend_context": {
    "design_system": {
      "framework": "tailwind",
      "version": "3.x",
      "style_guide_url": "https://...",
      "component_library": "shadcn/ui",
      "design_references": ["https://figma.com/..."]
    },
    "api_documentation": {
      "type": "openapi",
      "path": "docs/openapi.yaml",
      "url": "https://api.example.com/docs",
      "base_url": "https://api.example.com",
      "auth_method": "bearer"
    }
  }
}
```

#### iteration-plan.md Schema

```markdown
| Sprint | Name | Type | Objective | Priority | Dependencies | Status |
|--------|------|------|-----------|----------|-------------|--------|
| 0 | Setup | backend | Initialize project | CRITICAL | None | Pending |
| 1 | User API | backend | User CRUD endpoints | HIGH | Sprint 0 | Pending |
| 2 | Login UI | frontend | Login page component | HIGH | Sprint 1 | Pending |
| 3 | Dashboard | fullstack | Dashboard with data | HIGH | Sprint 1,2 | Pending |
```

### Sprint Splitting Principles

- Sprint 0: critical bugs/security or foundational setup
- Each sprint: 2-8 minutes of AI work
- Clear boundaries (full module or well-defined subset)
- Strictly ordered dependencies
- Each sprint produces testable output
- Each sprint includes unit tests for the code it produces
- **Sprint type classification**: Each sprint is classified as `frontend`, `backend`, or `fullstack`
- **Type-specific enhancements**: Frontend sprints get design and API context; backend sprints use standard workflow

## Example Workflow

### 1. Initialize with PRD

```bash
/sprint-init docs/prd.md
```

Sprint Flow will:
- Analyze PRD and split into sprints
- Classify each sprint type (frontend/backend/fullstack)
- Detect design system (if frontend sprints exist)
- Search for API documentation (if frontend sprints exist)
- Ask for design references and API doc location
- Generate `.sprint/config.json` with frontend context
- Create iteration plan with sprint types
- Generate type-specific handoffs

### 2. Review the Plan

```bash
/sprint-status
```

Shows:
- Sprint overview with types
- Progress percentage
- Current sprint
- Available commands

### 3. Execute Sprints

```bash
/sprint-run
```

For each sprint:
- Reads sprint type from iteration plan
- **Activates appropriate persona**:
  - Backend sprint → backend-developer persona
  - Frontend sprint → frontend-developer + design-specialist + api-integration-specialist
  - Fullstack sprint → fullstack-developer + design-specialist + api-integration-specialist
- **Loads persona-specific context**:
  - Backend: Standard context only
  - Frontend/Fullstack: Design system + API documentation + specialist guidance
- Runs clarification gate if needed
- Dispatches sub-agent with persona identity and specialized knowledge
- Validates completion using persona-specific quality standards
- Generates completion report with persona-specific validation checklists

## Requirements

- Claude Code CLI for the current plugin wrapper
- Codex, if you use the Codex wrapper described in `.codex/INSTALL.md`
- A PRD document in your project

## License

MIT
