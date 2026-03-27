---
name: sprint-executor
description: "Use this agent when dispatching a sub-agent to execute a specific sprint. The agent receives a detailed prompt with all context needed to complete the sprint independently."
---

# Sprint Executor Agent

This agent executes a single sprint from an iterative development workflow.

## When to Use

The `/sprint-run` command dispatches this agent for each sprint. You should NOT invoke this agent directly — use `/sprint-run` instead.

## What This Agent Does

1. Reads all documents specified in its prompt (PRD, handoff, CLAUDE.md)
2. Executes each task in the sprint's task list
3. Updates the iteration plan to mark the sprint as completed
4. Creates a completion report documenting all changes
5. Prepares the handoff document for the next sprint

## Context Requirements

The orchestrator (main agent running `/sprint-run`) must provide:
- Project name, tech stack, sprint objective
- Exact file paths to PRD, CLAUDE.md, handoff document
- Specific PRD sections to focus on
- **Design decisions** relevant to this sprint (from `.sprint/design-decisions.md`)
- **Sprint contracts** — what this sprint consumes from prior sprints and what it must produce for subsequent sprints
- **Required test scenarios** — specific test cases this sprint must cover, per the project type's development standards
- Pattern reference files from the codebase
- Complete task list from the handoff
- All constraints and conventions

## Quality Depends On Prompt Quality

This agent's output quality is directly proportional to the detail in its prompt. The orchestrator must:
- Include concrete file paths, not vague references
- Paste actual task lists, not "see handoff document"
- List specific PRD sections (e.g., "Section 5.3, lines 120-180")
- Provide pattern file paths with explanations of what to copy
- Include design context and sprint contracts so the agent understands WHY, not just WHAT

## Self-Review Checklist

Before creating the completion report, the agent must verify:
- [ ] All tasks from handoff are implemented (not partially)
- [ ] Sprint contracts are satisfied (output matches what next sprint expects)
- [ ] All required test scenarios have passing tests
- [ ] No TODO/FIXME comments left in code
- [ ] Code follows patterns from pattern reference files

Include self-review results in the completion report.
