---
type: Architecture
title: Daily Aggregate and Majesty Cake
description: The daily accountability aggregate combines daily walk, Arabic review, and Greek review into a compact score consumed by the Majesty Cake widget.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: mcp_server.py
  - resource: docs/linguistics
  - resource: docs/BEEMINDER_ACTIVITY_TRACKING.md
---

# Daily Aggregate

The aggregate reports three components: `daily_walk`, `arabic_review`, and `greek_review`. It returns satisfied count, total count, `all_clear`, score, language completion details, budget remaining, and system pulse.

A review status such as `laminar` or `cavitation` is a state signal from the review-pump machinery, not proof that the daily goal is complete. The widget and agent should use the explicit `satisfied`/`goal_met` fields. This is the right abstraction for a “what must happen today?” check; individual card mechanics remain in the language services.
