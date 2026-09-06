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

A review status such as `laminar` or `cavitation` is a state signal from the review-pump machinery (`mcp_server.py`), not proof that the daily goal is complete. These terms describe the review flow dynamics, not goal satisfaction:

- **Laminar**: The review is flowing smoothly — cards are being processed at a steady rate, with no backlog or blockage. This indicates the review-pump is operating within normal parameters (like `SNAPPY`/`LOCKSY` priority tiers in Beeminder's internal architecture — `docs/BEEMINDER_PRIORITY_LORE.md`).
- **Cavitation**: The review flow is disrupted — there is turbulence, blockage, or irregular processing (like a `WHALEY` or `UNDULY` task blocking the pipeline). This is a signal to check `runbooks/beeminder-emergency.md` for potential derailment risk or to verify that `satisfied`/`goal_met` fields still match the actual daily requirement.

The widget (`architecture/beeminder-majesty-cake.md`) and agent should use the explicit `satisfied`/`goal_met` fields to determine whether a goal is complete, not the review flow state (`laminar`/`cavitation`). The aggregate is the right abstraction for a “what must happen today?” check (`/mecris-orient` queries `architecture/narrator-context.md` which includes `daily_aggregate_status`); individual card mechanics remain in the language services (`docs/BEEMINDER_ACTUAL_TRACKING.md`).
