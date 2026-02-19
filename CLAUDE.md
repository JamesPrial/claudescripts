# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of bash wrapper scripts and agent definitions for invoking Claude Code CLI with pre-configured settings. All scripts are plain bash — no build, lint, or test tooling.

## Architecture

**Hub-and-spoke pattern** centered on `bin/claudebase`:

- `bin/claudebase` — Flag-based base wrapper that constructs and `exec`s a `claude` command. Accepts `-m` (model), `-p` (permission mode), `-t` (tools), `-s` (append system prompt), `-S` (replace system prompt), `-a` (agent), `-A` (agents JSON), `-n` (max turns), `-o` (output format). Defaults: model=sonnet, permission-mode=bypassPermissions. Always passes `-p` (print/non-interactive mode).
- `bin/askclaude`, `bin/ghcli`, `bin/push` — Thin wrappers that `exec claudebase` with specific flags and a system prompt. Each is ~5 lines.
- `agents/*.json` — One JSON file per agent (e.g. `git-ops.json`). Each file is a single-key object `{"name": {description, prompt, tools, model, color?}}`. Compose with `jq -s add agents/a.json agents/b.json` for `claudebase -A`.
- `.aliases` — Shell aliases (`haiku`, `sonnet`, `opus`, `danger`) and PATH setup for sourcing in user shell config.

## Adding a New Wrapper Script

Create a file in `bin/` following the existing pattern:

```bash
#!/usr/bin/env bash
exec claudebase -m <model> -t <tools> -s "<system prompt>" "$@"
```

Make it executable. The script inherits `claudebase` defaults (bypassPermissions, sonnet) for any flags not specified.

Scripts that load agents from JSON files should resolve them relative to the repo root. See `ghcli` for the pattern using `SCRIPT_DIR="$(cd "$(dirname "$0")/.." && pwd)"`.

## Adding a New Agent

Create `agents/<name>.json` with this structure:

```json
{
  "<name>": {
    "description": "One-line description of the agent's role.",
    "prompt": "System prompt with instructions and constraints.",
    "tools": ["Bash", "Read"],
    "model": "haiku",
    "color": "green"
  }
}
```

Required fields: `description`, `prompt`, `tools`, `model`. Optional: `color`.

## Conventions

- All wrapper scripts use `exec` to replace the shell process (no dangling bash parent).
- Wrapper scripts use `"$@"` to pass all positional args as the prompt, except scripts with a sensible default prompt which use `"${1:-default prompt}"`.
- Scripts assume `claudebase` is on PATH (via `.aliases` or user's PATH config).
- Agent definitions are individual JSON files in `agents/`, one per agent. Merge with `jq -s add` to pass multiple agents via `-A`.
- Model shorthand: `haiku`, `sonnet`, `opus` are accepted by claudebase and expanded to the appropriate `claude-*` model IDs.
- All invocations are non-interactive (print mode via `-p`). These scripts are for one-shot CLI usage, not interactive sessions.
