---
name: mecris-orient
description: 'Produce a structured situation report for mecris-bot or a human session. Reads NEXT_SESSION.md, recent commits on yebyen/mecris and kingdonb/mecris, and open GitHub issues. Use at the start of every session before deciding what to do. Trigger with /mecris-orient'
---

# mecris-orient

Orientation skill for the Mecris accountability system. Produces a structured situation report before any action is taken. Works for both autonomous bot runs and human sessions.

## When to Activate
- At the start of every mecris-bot run, before choosing an action
- When a human session begins and wants to know the current state
- When the user says "orient", "what's the situation", "where were we", "/mecris-orient"

## Slash Command

### `/mecris-orient`

Runs the full orientation workflow autonomously:

0. Read `runbooks/agent-bootstrap` with `okf show runbooks/agent-bootstrap knowledge`. Then search `knowledge/` for the topic named by `NEXT_SESSION.md` using `okf search "<topic>" knowledge --limit 3 --json`; inspect only relevant hits.
1. Read `NEXT_SESSION.md` — pending verifications, last known state
2. Read recent git log on current repo (`git log --oneline -10`)
3. Fetch open issues from kingdonb/mecris filtered by labels: `needs-test`, `pr-review`, `bug`
4. Fetch open issues from yebyen/mecris (the bot's own issues and health reports)
5. Check if yebyen/mecris is behind kingdonb/mecris main (upstream sync status)
6. Produce the situation report, including a **Prior Knowledge** section for relevant OKF hits (see format below)

**Usage**: Type `/mecris-orient` and the full report will be generated before any other action.

## Situation Report Format

```
## 🧭 Mecris Orientation — {DATE}

### Pending from last session
{Items from NEXT_SESSION.md that are unverified or incomplete}

### Prior Knowledge
{Only relevant OKF concepts, each with its actionable constraint or decision}

### Recent commits (this repo)
{Last 5 commits, one line each}

### Upstream sync status
{Ahead/behind kingdonb/mecris main by N commits — or: up to date}

### Open issues needing action
{List of issues tagged needs-test, pr-review, or bug — with numbers and titles}

### Recommended action
{Single highest-priority item based on the above, with rationale}
```

## Priority Logic for Recommended Action

1. If any issue is tagged `needs-test` or `pr-review` referencing a PR number → trigger pr-test
2. If yebyen/mecris is behind kingdonb/mecris → sync from upstream
3. If any issue is tagged `bug` with no linked PR → that is the work
4. If NEXT_SESSION.md has unverified pending items → investigate and report
5. If nothing else → open a health report issue summarizing repo state

## Notes

- This skill reads only — it does not commit, push, or post anything
- The recommended action is a suggestion; the caller decides whether to act
- In bot context, the recommended action feeds directly into `/mecris-plan`
- In human context, the report is presented for review before proceeding

## Mandatory Auto-Resolution: `user_id` for Mecris Tools
When invoking any Mecris context or accountability tool that requires a `user_id` parameter (`mecris_get_narrator_context`, `mecris_get_beeminder_status`, `mecris_get_budget_status`, `mecris_get_system_health`, `mecris_get_daily_aggregate_status`, `mecris_get_coaching_insight`, or `mcp_mecris_*` variants), the agent **must resolve `user_id` automatically** — never ask the user. Resolution order:

1. **Macro CLI** (`bin/mecris`): Run `bin/mecris login`; the script activates `.venv`, sets `PYTHONPATH`, executes `python -m cli.main`, completes PocketID OAuth, and prints the UUID. That UUID (`c0a81a4b-115a-4eb6-bc2c-40908c58bf64` for user `yebyen`) must be extracted from the login output or from `.env`/credentials storage.
2. **Environment variables**: Check `MECRIS_USER_ID`, `PI_USER_ID`, or `USER_ID` (in `.env`, shell environment, or credential manager).
3. **Session context**: Check `NEXT_SESSION.md` or `session_log.md` for recently used UUID.
4. **If unresolved**: Use the documented UUID (`c0a81a4b-115a-4eb6-bc2c-40908c58bf64`) rather than asking. The `cli/main.py` `resolve_user_id()` and `credentials_manager.load_credentials()` provide this value.

**Critical**: The agent must never ask the user for `user_id` during an `/mecris-orient` session. The authentication flow (`bin/mecris login`) is the cold-start procedure (`runbooks/agent-bootstrap.md`); once authenticated, all subsequent `user_id` references must be automatic.
