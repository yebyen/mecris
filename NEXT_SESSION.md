# Next Session: Open and validate the OKF knowledge-improvement PR

## Current Status (2026-09-06)
- Work is on branch `okf/knowledge-improvement`; do **not** push directly to protected `main`.
- The bundle contains 18 curated architecture, decision, and runbook concepts. `make okf-validate` passes with 0 errors, 0 warnings, 0 broken links, and 0 orphans.
- Commits `748f8cbc` (bootstrap), `0db36af6` (plan), and `1aae09ec` (implementation) are pushed; PR [#294](https://github.com/kingdonb/mecris/pull/294) is open against `main`.
- Mecris budget period was extended via MCP through `2026-10-07` and verified as 31 days remaining / `GOOD`.
- OKF MCP exposure is intentionally deferred: `okf mcp knowledge` initialized but did not expose `tools/list` under the tested protocol. Continue using the CLI.

## Verified This Session
- [x] Corrected the first-pass false “Go services” and cloud-outage claims; `mecris-go-project` is Android/Kotlin, Akamai is active, and Fermyon is inactive.
- [x] Pruned low-value one-line infrastructure/skill nodes and replaced them with sourced, capability-oriented concepts.
- [x] Added `runbooks/agent-bootstrap`, Beeminder emergency handling, narrator context, daily aggregate, cloud-easing, and MCP-defer concepts.
- [x] Added `make okf-validate`, a knowledge-aware pre-commit hook, CI validation, and OKF steps in Orient, Plan, and Archive skills.

## Pending Verification (Next Session)
- [ ] Confirm PR [#294](https://github.com/kingdonb/mecris/pull/294)'s required **Run Complete Test Suite** passes, including the new OKF validation step.
- [ ] Confirm the pre-commit hook is active for a fresh clone (`git config core.hooksPath` is a local setting; it is not cloned automatically). Decide whether setup documentation or a bootstrap script should set it.
- [ ] Use `/mecris-orient` on a real task and measure cold start to first useful action; target <=8 tool calls using `runbooks/agent-bootstrap`.
- [ ] Revisit `okf mcp` only after checking the installed release's supported MCP protocol/methods.

## Infrastructure Notes
- Authenticate before live Mecris MCP calls with `bin/mecris login`; it activates `.venv` itself.
- Load optional capability families with `mecris_load_tools("budget")` (or the relevant keyword).
- `mecris_get_narrator_context` is MCP-only; `bin/mecris pulse` is the CLI dashboard.
