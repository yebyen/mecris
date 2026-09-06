---
type: Decision
title: Deterministic Pi Status and Progressive Context
description: Make /status a native Pi command that bypasses the LLM, exposes only narrator context by default, infers identity from login credentials, and keeps OKF guidance out of always-on context.
generated: { by: agent/gpt-5.6-sol, at: 2026-09-06T23:34:00Z }
sources:
  - resource: .pi/extensions/mecris/index.ts
  - resource: .pi/extensions/mecris/README.md
  - resource: services/credentials_manager.py
  - resource: mcp_server.py
  - resource: AGENTS.md
  - resource: https://github.com/kingdonb/mecris/pull/296
---

# Decision

Pi now provides two deliberately different status paths:

- `/status` is a native extension command. It calls `get_narrator_context` directly with `{}`, bypasses the LLM, and formats five deterministic lines: budget, urgent Beeminder runway, process pulse, daily aggregate, and next action.
- `/mecris [focus]` remains model-mediated. It sends the live context to the selected model for a richer narrative interpretation.

This split keeps the fast path reliable for small local models while preserving an expressive path when interpretation is useful.

## Identity

The backend resolves identity through `services/credentials_manager.py`: an explicit argument first, then `~/.mecris/credentials.json` written by `bin/mecris login`, then optional `DEFAULT_USER_ID`. Normal single-user calls omit `user_id` entirely. The Pi bridge hides the optional `user_id` property from core read-only tool schemas so models do not deliberate over or expose identity values.

If no credentials or fallback exist, the backend returns `Authentication Required`; the recovery is `bin/mecris login`. Identity values must not be hardcoded in prompts, committed harness configuration, or knowledge articles.

## Progressive context and tools

`AGENTS.md` was reduced from 7,987 bytes to 1,449 bytes. It retains only always-needed rules and points to skills and OKF for deeper context. The Pi bridge now activates only `get_narrator_context` plus `mecris_load_tools`; the other forty tools remain registered but inactive until requested.

The model-driven `/status` prompt and redundant status skill were removed. Consequently `/status` does not receive AGENTS context, skill instructions, tool schemas, or the narrator payload in an LLM turn.

## FastMCP result shape

FastMCP returns narrator data under `structuredContent.result`. The first deterministic formatter mistakenly read `structuredContent` itself, producing `unknown` and `?` fields. The parser now unwraps `result` before formatting. If `/status` again displays all unknown values, inspect this wrapper first.

## Expected behavior and tradeoffs

- `/status` may take as long as `get_narrator_context` because the backend still gathers Beeminder, budget, aggregate, weather, presence, and health data. It immediately displays `Fetching Mecris status…` in Pi's footer, clears that indicator on completion or error, and spends no model tokens.
- `/status` intentionally reports only five lines and may omit useful nuance.
- `/mecris` is the richer, non-deterministic alternative and depends on the selected model's reasoning quality.
- Core tool schemas omit identity only in Pi's bridge; the Python functions retain optional `user_id` for explicit multi-user or administrative use.

## Validation

- Credential-only `get_narrator_context()` succeeded with no explicit identity or `DEFAULT_USER_ID` fallback.
- A live MCP result confirmed the `structuredContent.result` wrapper and returned budget `GOOD` with daily score `0/3`.
- The Pi extension loaded successfully via its normal extension path.
- Relevant credential, MCP, and narrator tests passed (37 tests).
- `okf validate knowledge --strict --drift` passed with zero errors, warnings, broken links, or orphans.

## Related Concepts

- [MCP Server](../architecture/mcp-server.md): Provides narrator context and credentials-first identity resolution.
- [Narrator Context](../architecture/narrator-context.md): Defines the unified live payload consumed by both status paths.
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): Documents login and identity recovery.
- [Mecris Gall Loop](../architecture/gall-loop.md): Full orientation remains separate from quick status.
