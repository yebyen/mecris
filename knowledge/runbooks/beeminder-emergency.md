---
type: Runbook
title: Beeminder Emergency Response
description: A short procedure for interpreting live goal risk and selecting the smallest action before derailment.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
stale_after: 2026-12-05
sources:
  - resource: docs/BEEMINDER_PRIORITY_LORE.md
  - resource: docs/BEEMINDER_ASYNC_LORE.md
  - resource: beeminder_client.py
---

stale_after: 2026-12-05
---


# Emergency Response

1. Query live Beeminder/narrator status; do not rely on an old OKF snapshot.
2. Sort by `derail_risk`, then smallest `safebuf`/runway, then deadline. “Derails tomorrow — act today” is the highest-priority actionable alert.
3. Confirm the goal's actual unit and activity source. A slug such as `bike` may feed the daily-walk path; the returned label is authoritative.
4. Take or record the smallest valid action that moves the goal, then re-query status.
5. Record only durable discoveries or decisions in OKF. Do not create a concept for every daily completion.
