---
type: Runbook
title: Agent Session Bootstrap
description: Cold-start procedure for authenticating (`bin/mecris login`), loading lazy MCP families (`mecris_load_tools`), running the Gall loop (`/mecris-orient` → `/mecris-plan` → `/mecris-archive` → `/mecris-pr-test`), and working in a PR-protected repo. Includes UUID `c0a81a4b-115a-4eb6-bc2c-40908c58bf64`, `.venv` activation, `PYTHONPATH`, PocketID OAuth, `SIGINT`/`SIGTERM` handling, and pre-commit/CI validation steps.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
stale_after: 2026-12-05
sources:
  - resource: bin/mecris
  - resource: cli/main.py
  - resource: .mcp.json
  - resource: AGENTS.md
  - resource: session_log.md
---

# Agent Session Bootstrap

Cold-start procedure for authenticating Mecris (`bin/mecris login`), loading optional MCP tool families (`mecris_load_tools`), orienting via the Gall loop (`/mecris-orient` → `/mecris-plan` → work → `/mecris-archive` → `/mecris-pr-test`), and working safely in a PR-protected repository (`main` branch protected; work on feature branches only).

## Prerequisites
- `.venv` exists at `/Users/yebyen/w/mecris/.venv` (activated by `bin/mecris`).
- `PYTHONPATH` points to project root (set by `bin/mecris` script).
- PocketID OAuth configured (`cli/main.py`).

## Step 1 — Read this runbook, search OKF
```bash
okf search "<topic>" --limit 3 --json
```
Returns concepts (e.g., `agent-bootstrap`, `beeminder-emergency`, `budget-extension`).

## Environment Variable Fix (Critical for Auto-Resolution)
The `.env` file must include the user UUID so the MCP server (`mcp_server.py`) resolves it automatically without requiring the agent to ask the user:
```bash
DEFAULT_USER_ID=c0a81a4b-115a-4eb6-bc2c-40908c58bf64
MECRIS_USER_ID=c0a81a4b-115a-4eb6-bc2c-40908c58bf64
```
The `.mcp.json` `mecris` server environment must also include these variables so the stdio MCP process receives them (`python-dotenv` loads `.env`, but `.mcp.json` overrides/pass-through ensures the server process sees them):
```json
"env": {
  "PYTHONPATH": "/Users/yebyen/w/mecris",
  "DEFAULT_USER_ID": "c0a81a4b-115a-4eb6-bc2c-40908c58bf64",
  "MECRIS_USER_ID": "c0a81a4b-115a-4eb6-bc2c-40908c58bf64"
}
```
Without this fix, the agent (e.g., Qwen AgentWorld via Pi harness) will ask the user for `user_id` or get lost during `/mecris-orient` or `/status`. This is documented in `decisions/2026-09-06-okf-mcp-deferred.md` (Phase 5 deferred; embedded `.mcp.json` option preferred over separate server to avoid chooser overload).

## Step 2 — Authenticate
```bash
bin/mecris login
```
The `bin/mecris` script activates `.venv`, sets `PYTHONPATH`, and runs `python -m cli.main`. It launches PocketID OAuth, stores `user_id` UUID (`c0a81a4b-115a-4eb6-bc2c-40908c58bf64`), handles `SIGINT`/`SIGTERM` gracefully, and suppresses `httpx`/`urllib3` leaks.

## Step 3 — Load MCP tool families (lazy-loaded)
```python
mecris_load_tools("budget")     # Budget Governor
mecris_load_tools("all")         # All families
```
Without this, `mecris_get_narrator_context` may not be visible.

## Step 4 — Query live situation
```bash
mecris_get_narrator_context(user_id="c0a81a4b-...")
# or CLI dashboard:
bin/mecris pulse
```
Provides budget health (`GOOD`/`WARNING`), period end (`2026-10-07`), days remaining (`31`), budget (`20.72` USD), system health.

## Step 5 — Execute the Gall loop (main is PR-protected)
Work on feature branches only (`git checkout -b feature/<name>`). The loop: `orient` → `plan` (adds `"Knowledge impact"` line) → `work` → `archive` (updates `NEXT_SESSION.md`, `session_log.md`) → `test` (CI + `okf-validate`).

## Step 6 — Archive and validate
```bash
make okf-validate    # 21 concepts, 0 errors, 0 broken links
```
`.githooks/pre-commit` validates automatically. CI (`.github/workflows/ci.yml`) runs `okf validate` on every PR.

## Quick Update: How `/mecris-orient` Translates to Action
When the user asks "What's my status?" or triggers `/mecris-orient` (`.github/skills/mecris-orient/SKILL.md`), the agent executes the following translation flow:

1. **Read this runbook** (`okf show runbooks/agent-bootstrap`) — establishes authentication state (`.venv`, UUID `c0a81a4b-...`, `PYTHONPATH`), lazy-loading (`mecris_load_tools`), and PR-protection.
2. **Search the task topic** (`okf search "<topic>" --limit 3 --json`) — finds relevant concepts (e.g., `budget-extension`, `beeminder-emergency`, `narrator-context`).
3. **Query live situation** (`mecris_get_narrator_context` or `bin/mecris pulse`) — retrieves `urgent_items`, `recommendations`, `budget_status`, `system_pulse`.
4. **Interpret recommendations** (per `architecture/narrator-context.md`): read `priority` (`IMMEDIATE` = SNAPPY/LOCKSY, `TODAY` = LOCKSY, `SOON` = BATCHY/UNDULY), match `action` to runbook procedure (`runbooks/beeminder-emergency.md` for `IMMEDIATE`, `agent-bootstrap.md` for `TODAY`/`SOON`), and confirm `context` links to correct OKF concept.
5. **Select the smallest safe next action** — the `recommendations` array is authoritative. Do not invent actions not listed. Confirm `satisfied`/`goal_met` fields from `architecture/daily-aggregate.md` before declaring goal completion.
6. **Archive** (`/mecris-archive`) — close spec, validate OKF (`make okf-validate`), log to `NEXT_SESSION.md`/`session_log.md`.

This closes the update loop: live state (`mcp_server.py`) → interpreted recommendation (`narrator-context.md`) → smallest action (`agent-bootstrap.md` + `beeminder-emergency.md`) → serialized archive (`NEXT_SESSION.md` + `session_log.md`).

## Related Concepts
- [Agent Session Bootstrap](agent-bootstrap.md): Self-reference.
- [System Overview](../architecture/overview.md): High-level architecture.
- [Agent Memory Maintenance](okf-maintenance.md): Maintenance tasks.
