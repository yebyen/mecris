---
type: Runbook
title: Beeminder Emergency Response
description: Procedure for interpreting live goal risk (derail_risk, safebuf, deadline), selecting the smallest valid action before derailment, and recording durable discoveries per Beeminder's Midnight Mandate.
generated: { by: agent/gpt-5.6-sol, at: 2026-09-06T23:07:26Z }
stale_after: 2026-12-05
sources:
  - resource: docs/BEEMINDER_ASYNC_LORE.md
  - resource: docs/BEEMINDER_LORE_CATALOG.md
  - resource: docs/BEEMINDER_ACTUAL_TRACKING.md
  - resource: beeminder_client.py
  - resource: architecture/beeminder-integration.md
---

# Emergency Response

When a Beeminder goal hits `derail_risk` of `CRITICAL` or `WARNING`, or `safebuf` (days until derailment) is `0`, act immediately. Do not rely on stale OKF snapshots — the live state is authoritative.

## Step 1 — Query live status
Query the Beeminder client directly:
```python
from beeminder_client import BeeminderClient
from services.credentials_manager import credentials_manager
client = BeeminderClient(user_id=credentials_manager.resolve_user_id())
status = await client.get_goal_status("bike")  # or any slug
```
The `BeeminderGoal` data structure (`beeminder_client.py`) provides:
- `slug`: Goal identifier (e.g., `bike`, `arabic-review`).
- `current_value`: Actual value submitted.
- `target_value`: Required value for the day.
- `safebuf`: Days until derailment (`int`).
- `deadline`: Derailment time (`datetime`).
- `derail_risk`: `CRITICAL` | `WARNING` | `CAUTION` | `SAFE`.
- `pledge`: Financial penalty for derailment (`float`).
- `runits`: Rate units (e.g., `"d"` for daily).

## Step 2 — Confirm the actual unit and source
A slug like `bike` may feed the daily-walk path (`docs/BEEMINDER_ASYNC_LORE.md`). The returned label (`title`) and rate units (`runits`) from the `BeeminderClient` are authoritative. Confirm:
```python
print(f"Goal: {goal.title} (slug: {goal.slug})")
print(f"Risk: {goal.derail_risk}, Buffer: {goal.safebuf}d, Deadline: {goal.deadline}")
print(f"Rate: {goal.rate} {goal.runits}, Target: {goal.target_value}")
```

## Step 3 — The Midnight Mandate (`docs/BEEMINDER_ASYNC_LORE.md`)
Derailments happen at **midnight local time**, not at the moment `safebuf` hits `0`. Between `0 hours` and actual derailment, there is a grace period. If the data has been submitted to the MCP tool (`record_groq_reading`, etc.), the user's obligation is met. The background sync workers (`Fellas`) handle propagation.

If a sync worker is offline, the failure is technical, not a failure of will. Use Beeminder's explainer feature if needed (`"I submitted the data to my robot, but the robot's shim crashed"`). They are reasonable.

## Step 4 — Select the smallest valid action
From `BEEMINDER_ASYNC_LORE.md`: *Put the data in the tool; let the fellas handle the rest.* The smallest action that moves the goal:
- Submit or confirm the data entry via the MCP tool.
- If the goal is derailing today (`deadline` today, `derail_risk` `CRITICAL`), submit the missing value immediately.
- Re-query status after submission. Do not expect instant sync — trust the `BeeminderClient` and `mcp_server.py` async pipeline.

## Step 5 — Record durable decisions
Only record durable discoveries or decisions in OKF (`knowledge/decisions/` or `knowledge/runbooks/`). Do not create a concept for every daily goal completion. The `session_log.md` records the action; `NEXT_SESSION.md` records the next planned action.

If the emergency reveals a new system failure (e.g., sync service offline, budget period expired), create or update a decision concept (`decisions/2026-09-06-critical-situation.md`) rather than a transient log entry.

## Related Concepts
- [Beeminder Goal Integration](../architecture/beeminder-integration.md): Live goal status and safety buffers.
- [Daily Aggregate and Majesty Cake](../architecture/daily-aggregate.md): X/Y score visualization from Beeminder + aggregate.
- [Agent Memory Maintenance](okf-maintenance.md): Post-emergency validation tasks (`make okf-validate`).
