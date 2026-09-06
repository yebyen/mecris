---
name: mecris-status
description: 'Minimal status update — reads bootstrap, queries narrator context with auto-resolved user_id, shows budget + beeminder + system pulse. No full OKF workflow. Trigger with /mecris-status or /status.'
---
# mecris-status

Minimal orientation for quick status checks. Does NOT run the full Gall loop or OKF search.

## When to Activate
- When user asks "/status", "what's my status?", "quick update", or needs a brief situation report.
- Before `/mecris-plan` when a rapid check is sufficient.

## Quick Workflow (≤3 steps)

### 1. Auto-resolve `user_id`
Resolve `user_id` from session/auth automatically (same rule as `/mecris-orient`):
```python
user_id = "c0a81a4b-115a-4eb6-bc2c-40908c58bf64"  # yebyen; resolved from bin/mecris login / session / .env
```
Never ask the user. The UUID is persistent for this user.

### 2. Query live situation (only this)
```python
mecris_get_narrator_context(user_id=user_id)
```
No `okf search`, no `NEXT_SESSION.md` read, no issue fetch.

### 3. Produce brief report (≤5 bullets)
Include ONLY:
- Budget health (`GOOD` / `WARNING` / `CRITICAL`) + days remaining + period_end.
- Beeminder alert (`derail_risk` + `safebuf` + `deadline`) from `urgent_items`.
- System pulse (`running`, `is_leader`, `last_error`).
- Daily aggregate score (`X/Y`) from `daily_aggregate_status`.
- Next immediate action (`IMMEDIATE` from `recommendations`).

No Prior Knowledge section, no commit log, no issue list, no OKF search results.

## Usage
```bash
/status        # Minimal report: budget + beeminder + pulse + action
```

## Reasoning Constraint (Critical)
Keep reasoning brief (≤3 lines of thought). Do NOT overthink — this is a quick check, not a full orientation. Focus only on:
1. Resolve user_id (automatic, never ask).
2. Call `mecris_get_narrator_context` (single call).
3. Extract exactly 5 bullets from the response (budget, beeminder, pulse, aggregate, action).
Do not search OKF, read session logs, or analyze relationships. The full `/mecris-orient` handles that.
