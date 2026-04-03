# sprint-init

Initialize Sprint Flow from a PRD document.

This is the Codex wrapper for the workflow defined by the Claude command at `plugins/sprint-flow/commands/sprint-init.md`.

## Required Behavior

- Analyze the PRD and create the `.sprint/` workflow files.
- Preserve the shared artifact model used by the Claude Code plugin.
- Set `runtime.host` to `codex`.
- Set `runtime.instruction_file_path` to `AGENTS.md` only when that file exists.
- If `AGENTS.md` does not exist, leave `runtime.instruction_file_path` empty.
- Do not substitute `CLAUDE.md` for Codex.

## Codex Adaptation Rules

- If Codex provides an interactive question capability, use it for batched clarification rounds.
- If not, ask one batched clarification round directly in the main session.
- Preserve the same project-level vs sprint-level ambiguity boundary as the Claude workflow.
- Preserve `.sprint/design-decisions.md`, `.sprint/config.json`, `.sprint/iteration-plan.md`, and handoff generation semantics.

## Source of Truth

Use `plugins/sprint-flow/commands/sprint-init.md` as the canonical workflow definition, but adapt any Claude-only tool wording to Codex capabilities.
