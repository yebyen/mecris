---
description: Serialize state: close spec, update NEXT_SESSION.md, append session_log.md, run make okf-validate, commit. References .github/skills/mecris-archive/SKILL.md.
---
Archive the session:

1. Read `.github/skills/mecris-archive/SKILL.md` (close spec issue, rewrite NEXT_SESSION.md, append session_log.md, validate OKF, commit).
2. Run `make okf-validate` (bundle: 22 concepts, 0 errors, 0 broken links, 0 stale).
3. Update `knowledge/decisions/` or `knowledge/runbooks/` only for durable discoveries.
4. Log delta (one line) to `session_log.md`; update `NEXT_SESSION.md` with pending verifications.
