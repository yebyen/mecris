---
type: Architecture
title: Beeminder Accountability Integration
description: Beeminder is the live source for goal runways, derail risk, daily activity, and emergency prioritization.
generated: { by: agent/cli, at: 2026-09-06T16:18:48Z }
sources:
  - resource: beeminder_client.py
  - resource: docs/BEEMINDER_ACTIVITY_TRACKING.md
  - resource: docs/BEEMINDER_PRIORITY_LORE.md
  - resource: mcp_server.py
---

# Beeminder Integration

The Beeminder client supplies goal status, safebuf/runway, deadline, current value, and derail risk. The narrator context turns that into urgent items and recommendations; agents should act on the smallest safe next action rather than treating OKF as a replacement for live Beeminder data.

The `bike` goal is also used by the daily walk status path, so do not infer semantics from the slug alone. Verify the returned label and activity source. For a goal with a one-day safebuf or `WARNING` risk, consult `runbooks/beeminder-emergency` and prioritize it before architecture work.

# Related Concepts
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Live goal risk is interpreted by the emergency procedure
- [Daily Aggregate and Majesty Cake](daily-aggregate.md): Beeminder activity feeds aggregate accountability
