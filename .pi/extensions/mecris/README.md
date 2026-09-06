# Mecris ↔ Pi bridge extension

Brings the Mecris MCP server into the [Pi coding agent](https://github.com/earendil-works)
as native Pi tools. See [`docs/PI_HARNESS_ROADMAP.md`](../../../docs/PI_HARNESS_ROADMAP.md)
for the full parity analysis and roadmap.

## Install

This directory is auto-discovered by Pi as a project-local extension
(`.pi/extensions/*/index.ts`) once you trust the project. From the repo root:

```bash
cd .pi/extensions/mecris && npm install    # one-time: fetch @modelcontextprotocol/sdk
```

Then just run `pi` in the repo. Or load it explicitly for a quick test:

```bash
pi -e ./.pi/extensions/mecris/index.ts
```

## Use

- `/status` — deterministic five-line live status. Calls `get_narrator_context` directly and does not invoke the model.
- `/mecris [focus]` — ask the model for a focused interpretation of live context.
- `/mecris-reconnect` — restart the bridge without a full `/reload`.
- Non-core tools are deferred; the model activates them via `mecris_load_tools`.

## Config (env vars)

| Var | Default | Purpose |
|---|---|---|
| `MECRIS_HOME` | repo root (3 levels up) | Mecris checkout location |
| `MECRIS_PYTHON` | `<home>/.venv/bin/python` | Python interpreter |
| `MECRIS_STDIO_SCRIPT` | `<home>/mcp_server.py` | canonical stdio entrypoint |
| `MECRIS_CORE_TOOLS` | `get_narrator_context` | comma-separated active-at-startup set |

## Identity resolution

The Python backend resolves the current user from `~/.mecris/credentials.json`, written by `bin/mecris login`. `DEFAULT_USER_ID` is an optional standalone fallback. Core read-only Pi tools hide the optional `user_id` field; callers normally use `{}`.

## Workflow prompts

Project prompts expose the Gall workflow: `/mecris-orient`, `/mecris-plan`, `/mecris-archive`, and `/mecris-pr-test`. Their canonical definitions live under `.github/skills/`.
