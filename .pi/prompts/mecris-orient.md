---
description: Produce a structured situation report before any action. References .github/skills/mecris-orient/SKILL.md.
---
Run the full orientation workflow before deciding what to do:

1. Read `runbooks/agent-bootstrap.md` via `okf show runbooks/agent-bootstrap knowledge`.
2. Search `knowledge/` for the session topic (`okf search "<topic>" --limit 3 --json`).
3. Query `mecris_get_narrator_context` (resolve `user_id` automatically — see `.github/skills/mecris-orient/SKILL.md`: never ask the user; use UUID `c0a81a4b-115a-4eb6-bc2c-40908c58bf64` from `bin/mecris login`, `.env`, or session context).
4. Read `NEXT_SESSION.md` for pending verifications.
5. Produce the situation report including Prior Knowledge, Recent Commits, Open Issues, System Health, Budget Status, and Recommended Action.
