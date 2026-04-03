# sprint-status

Show the current Sprint Flow status.

This is the Codex wrapper for the workflow defined by the Claude command at `plugins/sprint-flow/commands/sprint-status.md`.

## Required Behavior

- Read `.sprint/config.json` and `.sprint/iteration-plan.md`.
- If `.sprint/config.json` is missing, report that Sprint Flow is not initialized.
- Show overall progress, current sprint, and relevant next commands.
- If a sprint number is requested, show detailed information for that sprint.

## Source of Truth

Use `plugins/sprint-flow/commands/sprint-status.md` as the canonical workflow definition.
