# sprint-run

Execute Sprint Flow sprints sequentially.

This is the Codex wrapper for the workflow defined by the Claude command at `plugins/sprint-flow/commands/sprint-run.md`.

## Required Behavior

- Execute sprints strictly one at a time.
- Read and reuse the shared `.sprint/` artifacts.
- Run the clarification gate before implementation for each sprint.
- Preserve the clarification file contracts:
  - `.sprint/clarification-sprint-{N}.md`
  - `.sprint/sprint-{N}-clarifications.md`
- Preserve completion and handoff contracts.

## Codex Adaptation Rules

- If Codex supports delegated execution, use it.
- Otherwise run the sprint implementation in the main session using the same prompt contract.
- If Codex supports interactive questions, use them for a single batched clarification round.
- Otherwise pause and ask the user directly in the main session.
- Do not assume Claude-only primitives such as `Agent(...)` or `AskUserQuestion` exist.

## Source of Truth

Use `plugins/sprint-flow/commands/sprint-run.md` as the canonical workflow definition, but adapt Claude-specific tool references to Codex capability-based behavior.
