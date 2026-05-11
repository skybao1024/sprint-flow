# Codex Installation

Sprint Flow keeps a shared `.sprint/` workflow format across hosts. Codex support now uses the native Codex plugin package inside `plugins/sprint-flow`.

## What This Repository Ships

The Codex-distributed artifacts in this repository are:

```text
.agents/plugins/marketplace.json
plugins/sprint-flow/.codex-plugin/plugin.json
plugins/sprint-flow/skills/sprint-flow/SKILL.md
```

The Codex Skill stays thin. It routes user intent to the canonical workflow definitions at:

```text
plugins/sprint-flow/commands/*.md
```

## User Installation Model

Sprint Flow is published for Codex as a **Plugin** with a Skill entrypoint.

GitHub installation:

```bash
codex plugin marketplace add skybao1024/sprint-flow
```

This installs the marketplace directly from GitHub. End users do not need to clone the repository first once the Codex plugin files have been committed and pushed.

For Codex CLI versions that do not provide a separate plugin install command, enable the plugin in `~/.codex/config.toml`:

```toml
[plugins."sprint-flow@sprint-flow-marketplace"]
enabled = true
```

Restart Codex so it re-discovers the plugin metadata.

Codex discovers Sprint Flow as a Skill. Trigger it with the `$sprint-flow` mention:

```text
$sprint-flow initialize docs/prd.md
$sprint-flow show status
$sprint-flow run the next sprint
$sprint-flow complete the current sprint
```

In the Codex composer, type `$s` and select `sprint-flow (sprint-flow) [Skill]`. The un-namespaced Claude Code commands such as `/sprint-init` are not valid Codex commands. In the tested Codex CLI version, third-party plugin commands are not exposed as Sprint Flow slash commands in the TUI, even though the plugin Skill is loaded.

## Local Development Install

Before publishing, install from a local checkout:

```bash
codex plugin marketplace add /path/to/sprint-flow
```

When run from the repository root, this can also be:

```bash
codex plugin marketplace add .
```

## Legacy Skill Path

The old `.codex/skills/sprint-flow` layout existed before Codex had native plugin support. Do not duplicate command wrappers there. If you still have a local symlink such as `~/.codex/skills/sprint-flow`, update it to point at:

```text
plugins/sprint-flow/skills/sprint-flow
```

## Important Compatibility Notes

- Claude Code remains the canonical plugin distribution path via `.claude-plugin/`.
- Codex support is implemented through `.codex-plugin/plugin.json` plus `skills/sprint-flow/SKILL.md`.
- Both hosts reuse the same `.sprint/` artifacts and workflow semantics.
- The canonical command definitions stay in `plugins/sprint-flow/commands/`.
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
