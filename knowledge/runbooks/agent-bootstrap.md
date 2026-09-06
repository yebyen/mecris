---
type: Runbook
title: Agent Session Bootstrap
description: Cold-start procedure for authenticating Mecris, loading optional tools, orienting, and working safely in a PR-protected repository.
generated: { by: agent/cli, at: 2026-09-06T16:18:48Z }
stale_after: 2026-12-05
sources:
  - resource: bin/mecris
  - resource: cli/main.py
  - resource: .mcp.json
  - resource: AGENTS.md
  - resource: session_log.md
---

stale_after: 2026-12-05
---


# Bootstrap

1. Read this runbook, then search OKF for the task topic.
2. The Mecris MCP tools require authentication. Run `bin/mecris login`; the script activates `.venv`, sets `PYTHONPATH`, and runs `python -m cli.main`. Complete the PocketID browser redirect. The login output includes the UUID used as `user_id` when required.
3. Optional MCP families are lazy-loaded with `mecris_load_tools("budget")`, `mecris_load_tools("all")`, or the relevant capability keyword.
4. Query `mecris_get_narrator_context` for the live situation. `bin/mecris pulse` is the CLI dashboard. There is no `get-narrator-context` CLI verb.
5. Before coding, use the Gall loop: orient, plan, work, archive, test. Main is PR-protected; work on a branch and open a PR rather than pushing `main`.
6. At the end, run `make okf-validate` and commit the archive state.
