---
type: Architecture
title: Budget Governor Service
description: Python budget-governor logic enforces provider/bucket envelopes and supplies routing recommendations to agents.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: services/budget_governor.py
  - resource: docs/BUDGET_GOVERNOR_SPEC.md
  - resource: docs/BUDGET_GOVERNOR_GUIDANCE.md
  - resource: mcp_server.py
---

# Budget Governor

The implementation is Python in `services/budget_governor.py`, not Go. It tracks spend by provider/bucket, checks whether a proposed spend is allowed, records approved usage, and recommends a routing path. The MCP surface includes `mecris_get_budget_governor_status`, `mecris_budget_governor_check`, `mecris_budget_governor_record`, `mecris_budget_governor_recommend`, and `mecris_budget_governor_gate`.

The governor's envelope rules and bucket names are defined in `docs/BUDGET_GOVERNOR_SPEC.md`; consult that spec rather than duplicating limits here. The practical rule for an agent is: check before expensive work, prefer the recommended bucket, and record usage after the request.

# Related Concepts
- [Critical Situation: Budget Expired, Walk Needed, Reviewstack Derailing Tomorrow](../../decisions/2026-09-06-critical-situation.md): The snapshot recorded a budget-period correction
