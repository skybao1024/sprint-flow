# sprint-done

Complete the current sprint and prepare the next handoff.

This is the Codex wrapper for the workflow defined by the Claude command at `plugins/sprint-flow/commands/sprint-done.md`.

## Required Behavior

- Determine the target sprint from `.sprint/config.json` or the provided argument.
- Generate `.sprint/sprint-{N}-completion-report.md`.
- Update `.sprint/iteration-plan.md`.
- Prepare `.sprint/handoff-sprint-{N+1}.md` when more sprints remain.
- Update `.sprint/config.json` exactly as the shared workflow requires.

## Codex Adaptation Rules

- Preserve the same report and handoff semantics as the Claude workflow.
- Keep instructions host-neutral; do not assume Claude-only command runtime behavior.

## Source of Truth

Use `plugins/sprint-flow/commands/sprint-done.md` as the canonical workflow definition.
