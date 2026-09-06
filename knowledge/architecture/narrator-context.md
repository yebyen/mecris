---
type: Architecture
title: Narrator Context as Primary Agent Sensor
description: The narrator-context MCP response is the compact live situation report combining goals, budget, daily aggregate, system health, recommendations, and presence.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: docs/NARRATOR_CONTEXT_ARCHITECTURE.md
  - resource: mcp_server.py
  - resource: session_log.md
---

# Narrator Context

`mecris_get_narrator_context` is the preferred first live query after authentication. Its useful top-level fields are `summary`, `urgent_items`, `beeminder_alerts`, `goal_runway`, `budget_status`, `recommendations`, `daily_walk_status`, `daily_aggregate_status`, `system_pulse`, `presence`, and `budget_governor`.

Use `urgent_items` and `beeminder_alerts` to choose the next action; use `goal_runway` for the numbers behind that choice; use `system_pulse` and `presence` to avoid declaring a healthy system from stale data. `budget_status.days_remaining` and `period_end` are live operational values and must not be copied into durable architecture prose.

## Interpreting Recommendations (`recommendations`)
The `recommendations` array is the agent's primary action-selection signal. Each item has:
- `priority`: `IMMEDIATE` (SNAPPY tier — act before anything else; e.g., `derail_risk == CRITICAL` and `safebuf == 0`), `TODAY` (LOCKSY — critical background; submit missing data first), or `SOON` (BATCHY — regular maintenance; plan next session action).
- `action`: The smallest valid action (`submit_walk_data`, `record_review`, `check_budget_governor_gate`, etc.).
- `context`: Which OKF concept or runbook provides the procedure (`runbooks/beeminder-emergency.md`, `runbooks/agent-bootstrap.md`).

**How the agent picks the next action**: Read recommendations in priority order. Execute the `IMMEDIATE` item first (smallest action that moves the goal). Only proceed to `TODAY` after confirming no `IMMEDIATE` items remain. Use `SOON` items to populate `NEXT_SESSION.md`. Never invent an action not listed in recommendations — the live state (`mcp_server.py` + `BeeminderClient`) is authoritative.

## Interpreting Urgent Items (`urgent_items`)
`urgent_items` combines `beeminder_alerts` (live derail risk from `BeeminderClient.get_goal_status`) and `budget_governor` gate status (`budget_governor.md`). A `WARNING` or `CRITICAL` alert requires the `runbooks/beeminder-emergency.md` procedure. A budget gate warning (`budget_governor` gate triggered) requires checking `architecture/services/budget-governor.md` and reviewing the budget envelope (`docs/BUDGET_GOVERNOR_SPEC.md`).

**Priority mapping (from `docs/BEEMINDER_PRIORITY_LORE.md`)**:
- `IMMEDIATE` → `SNAPPY`/`LOCKSY` tier: User-facing or critical state updates. Block everything else.
- `TODAY` → `LOCKSY` tier: Critical background tasks (submit data, confirm sync). Must complete before session archive.
- `SOON` → `BATCHY`/`UNDULY`: Regular maintenance and next-session planning.

This mapping ensures high-priority tasks never starve behind background maintenance (`BeeminderClient` async pipeline handles propagation; the agent handles submission).

## System Pulse (`system_pulse`) and Presence (`presence`)
- `system_pulse`: A compact health signal (`GOOD`, `WARNING`, `DEGRADED`). If it shows `DEGRADED`, do not rely on cached data. Re-query live state directly (`bin/mecris pulse` or `mecris_get_narrator_context` with fresh `user_id`).
- `presence`: Indicates if the user has recently interacted (`present`, `idle`, `absent`). Use this to avoid declaring session success from stale data when the user is `absent`. If `absent`, archive should note that actions were taken without user confirmation.

Both fields are defensive: they prevent the agent from declaring a healthy system (`system_pulse == GOOD`) when the underlying sync service (`mecris-go-spin/sync-service`) or budget governor (`services/budget_governor.py`) is actually degraded.

## Budget Status Integration (`budget_status`)
`budget_status` includes `budget_health`, `days_remaining`, `period_end`, and `budget_governor_gate`. These are live values from the `budget_governor` Python service (`services/budget_governor.py`) and must not be copied into durable architecture concepts. Use them to inform recommendations (`recommendations`) and urgent items (`urgent_items`); do not treat them as architecture truth.

When `budget_health == CRITICAL` or `days_remaining < 0` (expired period), the emergency response procedure (`runbooks/beeminder-emergency.md`) takes priority. The budget extension decision (`decisions/2026-09-06-budget-extension.md`) records how this was recovered (MCP `mecris_update_budget` extended period to `2026-10-07`, restoring `31` days).

## Related Concepts
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Live goal risk interpretation procedure
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): Cold-start procedure that queries this context first

# Related Concepts
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Narrator alerts provide the emergency runbook inputs
