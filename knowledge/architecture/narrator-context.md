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

# Related Concepts
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Narrator alerts provide the emergency runbook inputs
