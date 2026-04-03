# Codex Installation

Sprint Flow keeps a shared `.sprint/` workflow format across hosts, but Codex installation/discovery stays separate from the Claude Code plugin wrapper.

## What This Repository Ships

The Codex-distributed artifact in this repository is the Skill at:

```text
.codex/skills/sprint-flow
```

It contains:

- `SKILL.md` — the Codex skill entrypoint
- `commands/*.md` — thin Codex wrappers around the canonical workflow docs in `plugins/sprint-flow/commands/*.md`

This is the open-source distribution unit for Codex users.

## User Installation Model

Sprint Flow is published for Codex as a **Skill**.

To install it on any machine:

1. Obtain this repository.
2. Install or place `.codex/skills/sprint-flow` into a Codex skills discovery path supported by your Codex installation.
3. Restart Codex so it re-discovers the skill metadata.

Use the Codex skills path documented by your Codex version as the source of truth for where the `sprint-flow` skill folder should live.

## Local Setup Example

If you are developing or testing locally, one practical setup is to link the repository's shipped skill folder into your local Codex skills path.

Example:

```bash
ln -s "$(pwd)/.codex/skills/sprint-flow" "$HOME/.codex/skills/sprint-flow"
```

This is a local installation example for development/testing, not the definition of the public distribution format.

## Important Compatibility Notes

- Claude Code remains the canonical plugin distribution path via `.claude-plugin/`.
- Codex support is implemented as a separate skill wrapper under `.codex/skills/sprint-flow`.
- Both hosts reuse the same `.sprint/` artifacts and workflow semantics.
- If Codex does not provide a Claude-equivalent delegation or interactive-question primitive, the Codex wrapper should fall back to:
  - single-sprint execution in the main session
  - clarification files plus a paused question round for the user

## Instruction File Mapping

The workflow keeps one canonical `runtime.instruction_file_path` field.

Set it explicitly by host:

- Claude Code → `CLAUDE.md`
- Codex → `AGENTS.md`

For Codex projects, the effective instruction file path written into `.sprint/config.json` should therefore be `AGENTS.md` when that file exists.

```json
{
  "runtime": {
    "host": "codex",
    "instruction_file_path": "AGENTS.md"
  }
}
```

If `AGENTS.md` does not exist, leave `runtime.instruction_file_path` empty rather than pointing Codex at another host's instruction file.
