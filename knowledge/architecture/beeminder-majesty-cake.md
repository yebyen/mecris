---
type: Architecture
title: Beeminder Majesty Cake Integration
description: The Majesty Cake widget visualizes daily accountability as X/Y goals satisfied, drawing from Beeminder goal data and the daily aggregate of walk, Arabic review, and Greek review.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: docs/BEEMINDER_ACTIVITY_TRACKING.md
  - resource: docs/linguistics
  - resource: mcp_server.py
---

# Majesty Cake Integration

The Majesty Cake widget is a visual accountability indicator that shows how many of the daily goals (walk, Arabic review, Greek review) have been satisfied today, expressed as “X/Y”.

It combines two data sources:
- **Beeminder goal data**: Provides the current value and safety buffer for each goal, determining whether the goal is satisfied for the day.
- **Daily aggregate**: Computes the count of satisfied goals from the language coaching service and the walk goal.

The widget updates in near‑real time via the Beeminder integration and the daily aggregate MCP operation.

## Related Concepts
## Related Concepts
- [Beeminder Goal Integration](../architecture/beeminder-integration.md): Supplies live goal status and safety buffers.
- [Daily Aggregate and Majesty Cake](../architecture/daily-aggregate.md): Computes the X/Y score displayed by the widget.
- [Mecris System Architecture Overview](overview.md): Majesty Cake is a key accountability widget shown in the system overview.
