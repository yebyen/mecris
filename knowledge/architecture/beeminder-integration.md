---
type: Architecture
title: Beeminder Accountability Integration
description: Beeminder is the live source for goal runways, derail risk, daily activity, and emergency prioritization.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: beeminder_client.py
  - resource: docs/BEEMINDER_ACTIVITY_TRACKING.md
  - resource: docs/BEEMINDER_PRIORITY_LORE.md
  - resource: mcp_server.py
---

# Beeminder Integration

The Beeminder client (`beeminder_client.py`) supplies goal status (`BeeminderGoal` structure: `slug`, `current_value`, `target_value`, `safebuf`, `deadline`, `derail_risk`, `pledge`, `rate`, `runits`), runway, and emergency prioritization. The narrator context turns that into urgent recommendations; agents should act on the smallest safe next action rather than treating OKF as a replacement for live Beeminder data.

## How Beeminder Serves (Internal Worker Architecture — `BEEMINDER_PRIORITY_LORE.md`)
These are Beeminder's internal background-worker tiers — important for integration design, not for emergency response:

- **SNAPPY**: Immediate user-waiting tasks (e.g., clicking "Sync" in the app).
- **LOCKSY**: Critical background-safe tasks (e.g., goal state updates, database writes).
- **BATCHY**: Regular maintenance (e.g., fetching 100 auto-sync goals).
- **WHALEY**: Resource-heavy, slow processing (e.g., historical re-imports).
- **UNDULY**: Low-priority "whenever" tasks.

The worker fallback pattern (check SNAPPY/LOCKSY before BATCHY) ensures high-priority tasks are never starved by background whales. When integrating with Beeminder (e.g., increasing polling frequency as a deadline approaches), apply this priority model: treat `CRITICAL` / `safebuf == 0` goals as **SNAPPY/LOCKSY** priority; completed goals can back off to **BATCHY/UNDULY**. This is a design principle for the `PriorityScanner` mentioned in `BEEMINDER_PRIORITY_LORE.md`, not an emergency response procedure.

The `bike` goal is also used by the daily walk status path, so do not infer semantics from the slug alone. Verify the returned label and activity source. For a goal with a one-day safebuf or `WARNING` risk, consult `runbooks/beeminder-emergency` and prioritize it before architecture work.

# Related Concepts
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Live goal risk is interpreted by the emergency procedure
- [Daily Aggregate and Majesty Cake](daily-aggregate.md): Beeminder activity feeds aggregate accountability
