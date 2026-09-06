---
description: Quick 3-step status — resolve UUID automatically, query live state, report 5 bullets. Never ask user for user_id.
---
Quick check: resolve UUID automatically (from session/auth), query live state, report 5 bullets. Skip full orientation workflow. (from `.env` `DEFAULT_USER_ID`, `.mcp.json` `MECRIS_USER_ID`, or `bin/mecris login` output: `auto-resolved from session/auth` for user `yebyen`).

Call `mecris_get_narrator_context(user_id=user_id)` once.

Report exactly 5 bullets (≤2 lines each, no extra reasoning):
1. Budget: health + days left + period_end (e.g., GOOD / 31 / 2026-10-07)
2. Beeminder: alert slug + derail_risk + safebuf + deadline (e.g., reviewstack / WARNING / 1 / 2026-09-07)
3. System pulse: running + is_leader + last_error (e.g., running: true / is_leader: false / last_error: null)
4. Daily aggregate: X/Y score + satisfied count (e.g., 0/3 — walk false, arabic false, greek false / laminar + cavitation)
5. Next IMMEDIATE action: from recommendations array (e.g., "Submit/review reviewstack data — derails tomorrow" / "Confirm PR #296 passes CI" / "Walk needed: bike goal false")

No Prior Knowledge section. No commit log. No OKF search. No issue fetch. Brief only.
