---
description: Produce a structured situation report before any action. References .github/skills/mecris-orient/SKILL.md.
---
Run the full orientation workflow before deciding what to do:

1. Read `runbooks/agent-bootstrap.md` via `okf show runbooks/agent-bootstrap knowledge`.
2. Search `knowledge/` for the session topic (`okf search "<topic>" --limit 3 --json`).
3. Query `mecris_get_narrator_context` without `user_id`; the backend uses the logged-in identity.
4. Read `NEXT_SESSION.md` for pending verifications.
5. Produce the situation report including Prior Knowledge, Recent Commits, Open Issues, System Health, Budget Status, and Recommended Action.
