---
type: Architecture
title: Daily Aggregate and Majesty Cake
description: The daily accountability aggregate combines daily walk, Arabic review, and Greek review into a compact score consumed by the Majesty Cake widget.
generated: { by: agent/gpt-5.6-sol, at: 2026-09-06T23:20:00Z }
sources:
  - resource: mcp_server.py
  - resource: services/review_pump_core.py
  - resource: docs/review_pump_core_spec.md
  - resource: docs/BEEMINDER_ACTIVITY_TRACKING.md
---

# Daily Aggregate

The aggregate reports three scored components: `daily_walk`, `arabic_review`, and `greek_review`. It returns `satisfied_count`, `total_count`, `all_clear`, `score`, component booleans, language details, budget remaining, and per-modality system pulse.

The widget and agents must use explicit `satisfied` or `goal_met` values for completion. Review-pump flow status is diagnostic context, not completion.

## Review-pump flow states

`services/review_pump_core.py` defines these states:

- `cavitation`: daily completions are below tomorrow's liability; the pump is starved.
- `laminar`: completions are between tomorrow's liability and the target flow rate; this is steady flow. Zero debt and zero liability is also vacuously laminar.
- `turbulent`: completions are at or above the target flow rate; the user is ahead of the required flow.

These terms are independent of Beeminder's SNAPPY/LOCKSY/BATCHY/WHALEY/UNDULY worker queues. Do not use either vocabulary as a substitute for the other's meaning.

## Consumers

- The Majesty Cake widget displays the `X/Y` score and `all_clear` state.
- Narrator context embeds the aggregate for model-mediated `/mecris` reports.
- Pi's deterministic `/status` command prints the score and satisfied count without interpreting flow states.

Individual card mechanics and target calculations remain in `services/review_pump_core.py`; this concept describes only the daily accountability boundary.

## Related Concepts

- [Beeminder Majesty Cake Integration](beeminder-majesty-cake.md): Visual consumer of the aggregate.
- [Narrator Context](narrator-context.md): Live payload containing aggregate status.
- [Deterministic Pi Status and Progressive Context](../decisions/2026-09-06-deterministic-status.md): Defines deterministic status formatting.
