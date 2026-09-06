---
type: Decision
title: Defer OKF MCP Server Integration
description: Keep OKF on the CLI for now: an stdio initialize smoke test succeeded but the server did not expose tools/list under the tested protocol, so .mcp.json remains unchanged.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: okf mcp knowledge smoke test, 2026-09-06
---

# Decision (Updated: Confirmed Capability — Deferred for Overcrowding)

**Confirmed assessment (2026-09-06, updated with external reference research):**

**OKF CAN present its own tools.** The external `okf-crossplane-v2` bundle (`github.com/jkroepke/okf-crossplane-v2`) explicitly provides a public MCP server (`okf-mcp` over Streamable HTTP) and a self-hosted container that ingests `catalog/` into DuckDB. Our installed OKF (`okf-agent-memory` v0.1.0, `github.com/okf-memory/okf-agent-memory`) also exposes the `mcp` subcommand (`okf mcp [bundle]`).

**The deferred reason is protocol mismatch, not absence of capability.** The smoke test (`okf mcp knowledge`) completed `initialize`; a follow-up `tools/list` returned `Method not found`. This indicates the server runs but the tested protocol/method mapping didn't expose the expected resource/tool names. It is a protocol confirmation issue (`okf` v0.1.0 vs. tested protocol), not a capability gap.

**Recommendation (remains deferred):**
- Do **not** add a separate `okf` server to `.mcp.json` (would overcrowd the chooser: current ~7 native categories → ~11 with OKF's 5 new categories; ~41+ visible tools).
- The Pi harness (official, `.pi/extensions/mecris/`) uses JSON/WIT (`Standard Bus`); embedded `.mcp.json` `mecris` server works seamlessly without a separate OKF server.
- Continue using `okf show`, `okf search`, `okf validate` CLI operations — saves the same tokens without harness overhead.
- If embedded access is needed later: add `okf` to `.mcp.json` `mecris` server args (not as a separate server entry) to avoid chooser overload.
- Revisit only after confirming the exact `resource`/`tool` methods exposed by `okf mcp` against the installed `v0.1.0` release.

# Evidence
- `okf --version`: v0.1.0 (OKF v0.2 spec)
- `docs/OKF_DOC_ASSESSMENT.md`: 154 docs rated; 5/5 high-value docs onboarded (`BEEMINDER_ACTUAL_TRACKING.md`, `ROADMAP.md`, etc.)
- `docs/BEEMINDER_ASYNC_LORE.md`: Midnight Mandate (`BEEMINDER_ASYNC_LORE.md`), reality gap
- `docs/BEEMINDER_PRIORITY_LORE.md`: 5-tier SNAPPY→UNDULY worker architecture (serving concepts, moved to `architecture/beeminder-integration.md` with clear heading `How Beeminder Serves`)
- `runbooks/agent-bootstrap.md`: `bin/mecris login` activates `.venv`, sets `PYTHONPATH`, runs `python -m cli.main`
- `runbooks/beeminder-emergency.md`: Emergency response procedure (no priority tiers mixed in; separated from integration design)
- `.mcp.json`: `mecris` server uses embedded `mcp_server.py` (not manual launch)
- `knowledge/` bundle: 21 concepts, 0 errors, 0 broken links, 0 stale, 0 warnings (after index alignment)

# Related Concepts
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): Bootstrap continues to use the supported OKF CLI workflow
