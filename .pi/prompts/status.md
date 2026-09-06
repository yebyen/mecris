---
description: Minimal status update — budget, beeminder alert, system pulse, action. References .github/skills/mecris-status/SKILL.md.
---
Run the minimal status update before deciding what to do:

1. Auto-resolve `user_id` (see `.github/skills/mecris-orient/SKILL.md`: never ask; use UUID `c0a81a4b-115a-4eb6-bc2c-40908c58bf64` from `bin/mecris login`, `.env`, or session).
2. Call `mecris_get_narrator_context(user_id=user_id)`.
3. Report (≤5 bullets): Budget health + days remaining + period_end; Beeminder alert (`derail_risk`, `safebuf`, `deadline`); System pulse (`running`, `is_leader`, `last_error`); Daily aggregate (`X/Y`); Next `IMMEDIATE` action (`recommendations` array, `runbooks/beeminder-emergency.md` for emergency).
