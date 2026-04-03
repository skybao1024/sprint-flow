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

- `sprint-init` — analyze a PRD and initialize `.sprint/`
- `sprint-run` — execute sprints sequentially
- `sprint-status` — show workflow progress
- `sprint-done` — complete the current sprint and prepare the next handoff

## Host Notes

- This directory is the formal Codex Skills entrypoint shipped by this repository.
- Claude Code plugin support remains separate under `.claude-plugin/`.
- This Codex wrapper reuses the same `.sprint/` semantics and command names.
- For Codex projects, `runtime.instruction_file_path` should point to `AGENTS.md` when that file exists.
- If the host does not support delegated execution or interactive questioning primitives, keep execution in the main session and use clarification files plus a paused user question round.

## Installation Boundary

This Skill is the Codex-specific distribution layer. It should stay thin:
- `SKILL.md` defines the Codex entrypoint
- `commands/*.md` define Codex-specific adaptation rules
- `plugins/sprint-flow/commands/*.md` remain the canonical workflow definitions

Do not duplicate the full workflow here.

## Command Files

Read the command wrappers in `commands/` and follow the one matching the user request.
