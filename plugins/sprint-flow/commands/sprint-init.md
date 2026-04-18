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

### 2.5. Split PRD into sprints and classify sprint types

After identifying major modules, split the PRD into sprints and classify each sprint's type:

**Sprint Type Classification Criteria**:

- **Frontend**: Sprint focuses on UI components, pages, styling, user interactions
  - Keywords: component, UI, page, view, form, button, modal, layout, style, responsive, design
  - File patterns: `*.jsx`, `*.tsx`, `*.vue`, `*.svelte`, `*.css`, `*.scss`, `components/*`, `pages/*`, `views/*`
  - PRD indicators: user interface, frontend, UI/UX, design, mockup, wireframe

- **Backend**: Sprint focuses on APIs, services, database, business logic
  - Keywords: API, endpoint, database, service, controller, model, schema, migration, auth, server
  - File patterns: `*.go`, `*.py`, `*.java`, `controllers/*`, `models/*`, `services/*`, `api/*`
  - PRD indicators: backend, API, database, server, service, endpoint

- **Fullstack**: Sprint involves both frontend and backend work
  - Has significant indicators from both frontend and backend categories

**Classification Process**:
1. For each planned sprint, analyze task descriptions and target files
2. Count frontend vs backend indicators (keywords, file patterns, PRD content)
3. Assign type based on dominant indicators:
   - If frontend_score > backend_score * 2: type = "frontend"
   - If backend_score > frontend_score * 2: type = "backend"
   - If scores are similar: type = "fullstack"
   - Default: type = "backend"
4. Store sprint type in iteration plan for later use

**Track Frontend Sprint Existence**:
- Set `has_frontend_sprints = true` if any sprint is classified as "frontend" or "fullstack"
- Set `has_backend_sprints = true` if any sprint is classified as "backend" or "fullstack"
- These flags control whether to execute frontend context discovery in Step 3.5

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

### 3.5. Conditional Frontend Context Discovery

**ONLY execute this step if `has_frontend_sprints = true`** (i.e., at least one sprint is classified as "frontend" or "fullstack"). If the project has no frontend sprints, skip to Step 4.

#### 3.5a. Design System Detection

Search for design system indicators in the project:

1. **Check package.json** for UI framework dependencies:
   - Tailwind CSS: `tailwindcss`
   - Material-UI: `@mui/material`, `@material-ui/core`
   - Ant Design: `antd`
   - Chakra UI: `@chakra-ui/react`
   - shadcn/ui: Check for `components.json`
   - Bootstrap: `bootstrap`, `react-bootstrap`

2. **Check for config files**:
   - `tailwind.config.js`, `tailwind.config.ts`
   - `theme.js`, `theme.ts`
   - `components.json` (shadcn/ui)

3. **Grep for design system imports** in existing component files (if any):
   - Search for common import patterns in `*.jsx`, `*.tsx`, `*.vue` files

4. **Ask user for confirmation and additional context**:
   - Confirm detected design system
   - Request design reference URLs (Figma, style guide, mockups)
   - Ask for component library preferences

#### 3.5b. API Documentation Discovery

Search for API documentation in the project:

1. **Search for Swagger/OpenAPI files**:
   - Common paths: `swagger.json`, `openapi.yaml`, `openapi.json`, `api-docs.json`
   - Common directories: `docs/`, `api/`, `spec/`

2. **Check README and config files** for API documentation URLs

3. **Ask user for API documentation details**:
   - Confirm API documentation location (file path or URL)
   - Request API base URL for development environment
   - Ask about authentication method (bearer token, API key, OAuth2, etc.)

#### 3.5c. Store Frontend Context in config.json

Add `frontend_context` object to `.sprint/config.json` with discovered information:

```json
"frontend_context": {
  "design_system": {
    "framework": "<tailwind|material-ui|ant-design|chakra-ui|bootstrap|custom|none>",
    "version": "<version from package.json>",
    "style_guide_url": "<user-provided URL>",
    "component_library": "<shadcn/ui|custom|none>",
    "design_references": ["<figma_url>", "<mockup_url>"]
  },
  "api_documentation": {
    "type": "<swagger|openapi|custom|none>",
    "path": "<relative path to API doc file>",
    "url": "<API documentation URL>",
    "base_url": "<API base URL for development>",
    "auth_method": "<bearer|api-key|oauth2|none>"
  }
}
```

#### 3.5d. Add Frontend Context to design-decisions.md

If frontend sprints exist, append this section to `.sprint/design-decisions.md`:

```markdown
## Frontend Development Context

### Design System
**Framework**: <framework>
**Version**: <version>
**Component Library**: <library>
**Style Guide**: <url>

### Design Principles
- Follow design system patterns and conventions
- Maintain consistent spacing, typography, and color usage
- Ensure responsive design across breakpoints
- Prioritize accessibility (WCAG 2.1 AA minimum)

### Accessibility Requirements
- WCAG 2.1 AA compliance
- Keyboard navigation support for all interactive elements
- Screen reader compatibility with proper ARIA labels
- Color contrast ratios meet accessibility standards
- Focus indicators visible and clear

### API Integration
**API Documentation**: <type> at <path or url>
**Base URL**: <base_url>
**Authentication**: <auth_method>

### API Contract Rules (Frontend)
- All API endpoints must match documented contracts exactly
- Request and response schemas must be validated against documentation
- No invented endpoints or modified contracts allowed
- API changes or discrepancies must be escalated to backend team
- Document all API integration assumptions in completion reports
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
  "has_frontend_sprints": <true|false>,
  "has_backend_sprints": <true|false>,
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
  },
  "frontend_context": {
    "design_system": {
      "framework": "<tailwind|material-ui|ant-design|chakra-ui|bootstrap|custom|none>",
      "version": "<version>",
      "style_guide_url": "<url>",
      "component_library": "<library>",
      "design_references": ["<url1>", "<url2>"]
    },
    "api_documentation": {
      "type": "<swagger|openapi|custom|none>",
      "path": "<path>",
      "url": "<url>",
      "base_url": "<base_url>",
      "auth_method": "<bearer|api-key|oauth2|none>"
    }
  }
}
```

**Note**: The `frontend_context` field should only be included if `has_frontend_sprints = true`. If the project has no frontend sprints, omit this field entirely.

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

| Sprint | Name | Type | Objective | Priority | Dependencies | Status |
|--------|------|------|-----------|----------|-------------|--------|
| 0 | <name> | <frontend\|backend\|fullstack> | <objective> | CRITICAL/HIGH/MEDIUM | None | Pending |
| 1 | <name> | <frontend\|backend\|fullstack> | <objective> | ... | Sprint 0 | Pending |

## Sprint Details

### Sprint 0: <Name>

**Type**: <frontend\|backend\|fullstack>
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

{if sprint_type == "frontend" or sprint_type == "fullstack"}
## Design Context (Frontend Sprint)

**Design System**: {framework} {version}
**Component Library**: {library}
**Style Guide**: {url}
**Design References**: {figma/mockup links}

### Design Requirements
- Follow design system patterns and conventions from {framework}
- Match existing component styles and patterns
- Implement responsive design for all breakpoints
- Ensure WCAG 2.1 AA accessibility compliance
- Use design references for visual guidance

## API Documentation (Frontend Sprint)

**API Base URL**: {base_url}
**Authentication**: {auth_method}
**API Documentation**: {type} at {path or url}

### Available Endpoints for This Sprint

{List relevant API endpoints with request/response schemas extracted from API documentation}

#### Example Format:
**GET /api/users**
- **Description**: Retrieve user list
- **Authentication**: Bearer token required
- **Request Parameters**: 
  - `page` (number, optional): Page number
  - `limit` (number, optional): Items per page
- **Response Schema**:
  ```json
  {
    "data": [{"id": 1, "name": "string", "email": "string"}],
    "total": 100,
    "page": 1
  }
  ```

### Frontend API Integration Rules
- MUST use documented endpoints exactly as specified
- MUST match request/response schemas from documentation
- MUST NOT invent endpoints or modify contracts
- Document any API discrepancies in completion report
- Escalate API issues to backend team
{endif}

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
