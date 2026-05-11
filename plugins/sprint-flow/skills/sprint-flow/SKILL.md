---
name: sprint-flow
description: Codex wrapper for the Sprint Flow iterative development workflow. Use when the user wants to split a PRD into sprints, execute sprints sequentially, review sprint progress, or complete the current sprint while reusing the shared `.sprint/` workflow artifacts.
---

# Sprint Flow

Sprint Flow is a Codex wrapper for the same iterative workflow shipped to Claude Code.

It keeps one shared artifact model across hosts:
- `.sprint/config.json`
- `.sprint/iteration-plan.md`
- `.sprint/design-decisions.md`
- sprint handoff, clarification, and completion-report files

## Available Commands

- `sprint-init` - analyze a PRD and initialize `.sprint/`
- `sprint-run` - execute sprints sequentially
- `sprint-status` - show workflow progress
- `sprint-done` - complete the current sprint and prepare the next handoff

## Command Routing

Use the canonical command definitions in `../../commands/`, relative to this Skill directory:

- `../../commands/sprint-init.md`
- `../../commands/sprint-run.md`
- `../../commands/sprint-status.md`
- `../../commands/sprint-done.md`

Apply the Codex adaptation rules in this Skill when a canonical command mentions Claude-only behavior.

## Host Notes

- This directory is the Codex plugin Skill entrypoint.
- Claude Code plugin support remains separate under `.claude-plugin/`.
- This Codex wrapper reuses the same `.sprint/` semantics and command names.
- For Codex projects, `runtime.instruction_file_path` should point to `AGENTS.md` when that file exists.
- If the host does not support delegated execution or interactive questioning primitives, keep execution in the main session and use clarification files plus a paused user question round.
- Set `runtime.host` to `codex` when initializing or updating `.sprint/config.json`.
- Do not substitute `CLAUDE.md` for Codex.
- If Codex supports delegated execution, execute sprints one at a time through Codex delegation. Otherwise run the sprint implementation in the main session using the same prompt contract.
- If Codex supports interactive questions, use one batched clarification round. Otherwise pause and ask the user directly in the main session.

## Installation Boundary

This Skill is the Codex-specific distribution layer. It should stay thin:
- `SKILL.md` defines the Codex entrypoint
- `../../commands/*.md` remain the canonical workflow definitions

Do not duplicate the full workflow here.

## Source of Truth

Read the matching canonical command file in `../../commands/` and adapt Claude-specific tool references to Codex capability-based behavior.
