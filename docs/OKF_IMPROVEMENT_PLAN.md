# OKF Improvement Plan — Bang-for-Buck Edition

**Author:** Claude (planning pass, 2026-09-06)
**Executor:** auto model router (any capable model; follow steps literally)
**Baseline:** 18 concepts, `okf validate --strict --drift` clean, committed locally on `main` as `748f8cbc` (unpushed — main is PR-protected).

---

## 0. Honest Assessment of the Current Bundle

The first pass was a *scaffold*, not a knowledge base. Problems, in order of severity:

| # | Problem | Evidence | Cost if unfixed |
|---|---------|----------|-----------------|
| 1 | **Fabricated facts.** `architecture/go-services` claims Budget Governor, Coaching, Auth "run as Go services." They are Python (`services/*.py`). `mecris-go-project` is the **Android/Kotlin** app. `mecris-go-spin` is mostly **Rust** (`*-rs`) + one Go `sync-service`. `mecris-go/` is a disambiguation README. | `head mecris-go-project/README.md`, `ls mecris-go-spin`, `ls services/` | Agents trust OKF *more* than code; wrong OKF is worse than no OKF. |
| 2 | **Empty bodies.** Every concept is frontmatter + auto-generated "Related Concepts". Zero prose, zero sources, zero "how do I actually do X." | `cat knowledge/architecture/services/budget-governor.md` | `okf search` hits return a one-liner; agent still has to read code. No token savings realized. |
| 3 | **Stale-on-arrival claims.** `iron-heart` says Akamai/Fermyon "currently experiencing outages." `DEPLOYMENT.md` says Akamai is **ACTIVE & AUTHORITATIVE**, Fermyon is **deprovisioned**. | `head mecris-go-spin/DEPLOYMENT.md` | Same as #1. |
| 4 | **Low-value nodes.** Four `infrastructure/*-status` concepts are one-line descriptions of skill names already listed in `AGENTS.md`. They are the "PriceMapper.cs" of this bundle. | Prem Prakash article, Tier 3 | Retrieval noise; padding the graph with what the skill index already says. |
| 5 | **No provenance.** `sources:` is absent everywhere. `generated.by` is the generic `agent/cli`. | any concept file | Violates the skill's own contract (§1.6). Can't tell inference from fact. |
| 6 | **Not wired into the loop.** `/mecris-orient`, `/mecris-archive` don't mention `okf`. Nothing forces validation before commit. | `.github/skills/mecris-orient/SKILL.md` | Bundle rots silently. |
| 7 | **The one discovery that mattered isn't recorded.** "Load venv → `bin/mecris login` → then MCP tools work" cost ~20 tool calls to figure out. It is not in the bundle. | this session | Next session pays the same 20 calls. |

**Verdict:** OKF *will* be useful, but only after the bundle contains things an agent can't get faster by `grep`. Current bundle: ~0 net token savings. Target after this plan: orient-phase context load drops from "read 5 docs" to "3 `okf show` calls."

---

## 1. Guiding Rules for the Executor

1. **Every concept body must answer: "What would I have to read/run to learn this without OKF?"** If the answer is "one `ls`", delete the concept.
2. **`sources:` is mandatory.** Point at repo paths or URLs. If a claim has no source, prefix it with `(inferred)`.
3. **Prefer `okf update` over `okf create`.** Rewriting body text: edit the `.md` directly *below* the frontmatter, then run `okf validate`. The CLI only manages frontmatter/index/log.
4. **1 file = 1 capability**, not 1 file = 1 module. Merge, don't split.
5. **Run `okf validate knowledge --strict --drift` after every phase.** Non-zero = stop and fix.
6. **Set `generated.by`** to the actual model string (e.g. `gpt-oss-120b/pi`), never `agent/cli`. Edit frontmatter by hand if the CLI doesn't expose it.
7. Do **not** touch `AGENTS.md`'s OKF block except where Phase 4 says so.

---

## 2. Phases (do in order; each is independently shippable)

### Phase 1 — Stop the Bleeding (fix what's wrong) · ~15 min

**1a. Fix `architecture/go-services` → rename concept to `architecture/edge-and-clients`**

There is no `okf rename`. Do: `git mv knowledge/architecture/go-services.md knowledge/architecture/edge-and-clients.md`, then edit frontmatter `id`/`title`, then grep-fix inbound links in `architecture/overview.md` and `architecture/index.md`. Body:

```
# Edge Runtimes & Clients (the misnamed "mecris-go" family)

## What each directory actually is
- `mecris-go/` — disambiguation README only. Not code.
- `mecris-go-project/` — **Android app (Kotlin, Gradle).** Phase-1 vertical slice. PocketID auth via AppAuth, Health Connect walk ingestion, WorkManager background workers (`WalkHeuristicsWorker`, `DelayedNagWorker`).
- `mecris-go-spin/` — **Spin/WASM edge components.** Mostly Rust (`goal-type-rs`, `majesty-cake-rs`, `nag-engine-rs`, `review-pump-rs`), one Go service (`sync-service`), plus `arabic-skip-counter`, `schema.sql`.

## Deployment truth (per mecris-go-spin/DEPLOYMENT.md)
- **Akamai Functions** (`*.fwf.app`) — ACTIVE & AUTHORITATIVE. Neon Postgres, encrypted Twilio tokens, WhatsApp Utility Templates.
- **Fermyon Cloud** — INACTIVE, deprovisioned. Failed at WASM instantiation due to socket/capability limits in shared runtime.
- Local Python MCP (`mcp_server.py`) is the *primary* backend for interactive sessions (see decisions/2026-06-cloud-easing).

## Why "go" in the name
Historical: "Mecris-Go" = "Mecris on the go" (mobile), not Golang. (inferred from README titles)
```
`sources: [mecris-go/README.md, mecris-go-project/README.md, mecris-go-spin/DEPLOYMENT.md, ARCHITECTURE.md]`

**1b. Delete the three wrong relations** from `edge-and-clients` → `services/*`. Remove the lines from the "Related Concepts" section; the CLI has no `unrelate`. Then re-add the correct one:
`okf relate architecture/edge-and-clients architecture/data/neon-db knowledge --desc "Akamai edge components and Android sync write to Neon via sync-service"`

**1c. Fix `infrastructure/iron-heart`** description + body to match DEPLOYMENT.md (Akamai active, Fermyon dead). Consider merging into `edge-and-clients` and deleting `iron-heart`; the metaphor is preserved in `philosophy`. **Recommended: merge & delete.**

**1d. Delete the four Tier-3 nodes:** `infrastructure/prometheus-status`, `alertmanager-install`, `flux-status`, `kubeconfig-setup`. Remove their lines from `architecture/overview.md` and `infrastructure/index.md`. If `infrastructure/` is now empty after 1c, delete the directory and its `index.md`. Replace with **one** concept in Phase 2 (`runbooks/observability`) only if you can source real Prometheus/Flux config in the repo — otherwise skip entirely.

**1e. Fix `services/budget-governor`** — it's Python. Body should point to `services/budget_governor.py`, `docs/BUDGET_GOVERNOR_SPEC.md`, `docs/BUDGET_GOVERNOR_GUIDANCE.md`. Include the **bucket names** and the 5%/5% envelope rule (grep the spec). Include the MCP tool names: `mecris_get_budget_governor_status`, `_check`, `_record`, `_recommend`, `_gate`.

Validate. Commit as `fix(okf): correct fabricated go-services facts, prune tier-3 nodes`.

---

### Phase 2 — Add the Concepts That Pay Rent · ~45 min

Create these. Each must have a real body (≥ 10 lines) and `sources:`. Ordered by expected re-read frequency.

**2a. `runbooks/agent-bootstrap`** — *the single highest-value node.*
```
# Agent Session Bootstrap (do this before any Mecris MCP call)

1. Mecris MCP tools return {"error":"Authentication Required"} until you log in.
2. Login: `bin/mecris login` (script self-activates .venv, sets PYTHONPATH, runs `python -m cli.main`).
   Opens browser → PocketID at metnoom.urmanac.com → redirect to localhost:54321.
   Success prints: "Logged in as: yebyen (<uuid>)". That UUID is `user_id` for tools that need it.
3. Non-default MCP tools (budget, governor, GDPR) are hidden until `mecris_load_tools("budget")` etc.
4. Quick health: `bin/mecris pulse` (rich TUI dashboard) or `mecris_get_narrator_context` (JSON).
5. CLI subcommands: login | presence | internal | pulse | nag. There is NO `get-narrator-context` CLI verb; that's MCP-only.
6. `main` branch is PR-protected: "Changes must be made through a pull request" + required check "Run Complete Test Suite". Never `git push origin main`; branch + PR.
```
`sources: [bin/mecris, cli/main.py, .mcp.json, "session: 2026-09-06 OKF setup"]`
Type: `Runbook`.

**2b. `decisions/2026-06-cloud-easing`** — extract from `ARCHITECTURE.md` §"The Great Cloud Easing". Decision: pivot to local Python MCP as primary. Status: reversible. Open question: bisect deployment history. Link → `edge-and-clients`, `mcp-server`.

**2c. `architecture/data-model`** — *replace* `data/neon-db` (keep the id, rewrite body). Source `mecris-go-spin/schema.sql` + `migrations/`. List the tables the GDPR tool enumerates (from `mecris_delete_user_data` description): `users`, `token_bank`, `walk_inferences`, `language_stats`, `goals`, `message_log`, `usage_sessions`, `autonomous_turns`, `budget_tracking`, `scheduler_election`. Note: `token_bank` has no CASCADE. Note the multi-tenancy rule (every query bounded by `user_id`, per ROADMAP Goal 0).

**2d. `architecture/narrator-context`** — the JSON shape of `get_narrator_context` is the agent's primary sensor. Document its top-level keys and what each urgent/alert string means. Source: `docs/NARRATOR_CONTEXT_ARCHITECTURE.md` + a live call.

**2e. `architecture/gall-loop`** — *rewrite body*. Map each skill to its file in `.github/skills/`, what it reads, what it writes:
- orient → reads `NEXT_SESSION.md`, recent commits, open issues → writes nothing
- plan → writes a GitHub spec issue (intent / because / validation)
- archive → closes spec, updates `NEXT_SESSION.md`, appends `session_log.md`
- pr-test → dispatches workflow, polls
Add the sibling skills that exist but weren't in AGENTS.md: `mecris-bonsai`, `mecris-rebase`, `parity-arbitration`, `release-workflow`, `beeminder-audit`, `chore-*`.

**2f. `runbooks/beeminder-emergency`** — from `docs/BEEMINDER_PRIORITY_LORE.md` + `BEEMINDER_ASYNC_LORE.md`. What `safebuf`, `derail_risk`, `runway` mean; what "Derails tomorrow – act today" requires; which goals are pledge-bearing (`ellinika` $5, `elleniki` $5 — from live status).

**2g. `architecture/daily-aggregate`** — the 3-goal Majesty Cake (`daily_walk`, `arabic_review`, `greek_review`), what "laminar" vs "cavitation" mean for review pumps, issue #170. Source `mcp_server.py` + `docs/linguistics/`.

**2h. Rewrite bodies** of `mcp-server`, `beeminder-integration`, `coaching-service`, `auth-service`, `philosophy`, `overview` with real content + `sources:`. For `mcp-server`: list tool *categories* and the `mecris_load_tools` lazy-loading pattern, not all 50 tools. For `auth-service`: PocketID v2.13, 1h access / 30-day sliding refresh, single-use rotation, singleton repository pattern (from `session_log.md` 2026-08-20 — this is a hard-won lesson).

**2i. Delete or fill `specialized-skills`** — it duplicates AGENTS.md. Recommend delete; `gall-loop` (2e) now covers skills.

Validate. Commit as `feat(okf): add runbooks, data model, decisions; fill concept bodies`.

---

### Phase 3 — Make It Self-Maintaining · ~20 min

**3a. Makefile target** (append):
```make
okf-validate:
	okf validate knowledge --strict --drift

okf-check-drift:
	@okf validate knowledge --strict --drift --json | python3 -c 'import sys,json; d=json.load(sys.stdin); sys.exit(1 if d.get("errors") or d.get("warnings") else 0)'
```

**3b. Pre-commit hook** — `.githooks/pre-commit` (or extend existing): if `git diff --cached --name-only | grep -q '^knowledge/'` then `make okf-validate`. Document `git config core.hooksPath .githooks` in `runbooks/agent-bootstrap`.

**3c. CI** — add a step to the existing "Run Complete Test Suite" workflow (find it under `.github/workflows/`): install `okf` (check if there's a GitHub release / `go install`; if not, skip CI and rely on the hook). Only fail on `knowledge/` changes.

**3d. Stale dates** — add `stale_after:` frontmatter to volatile concepts: `runbooks/agent-bootstrap` (+90d), `decisions/2026-06-cloud-easing` (+180d), `beeminder-emergency` (+90d). `okf validate` reports stale count.

Commit as `chore(okf): validate on commit and CI, mark volatile concepts stale_after`.

---

### Phase 4 — Wire Into the Gall Loop · ~15 min

Edit the four skill files in `.github/skills/mecris-*/SKILL.md` (these are the canonical copies; `.claude/skills/` is a mirror — check `.gitmodules` / symlinks before editing both).

**4a. `mecris-orient`** — add step 0:
> Before reading NEXT_SESSION.md: `okf show runbooks/agent-bootstrap knowledge` and `okf search "<topic from NEXT_SESSION title>" knowledge --limit 3 --json`. Include any hits under a "Prior Knowledge" heading in the sitrep.

**4b. `mecris-plan`** — add:
> If the spec touches architecture/data/auth/deploy, name the OKF concept(s) it will update in the issue body under "Knowledge impact".

**4c. `mecris-archive`** — add before the session_log append:
> Run the End-of-Task checklist from AGENTS.md OKF block. Create/update concepts for any decision made. Run `make okf-validate`. Append the `knowledge/log.md` delta summary (one line) to the session_log entry.

**4d. `AGENTS.md` OKF block** — one addition only, under "Essential Memory Commands":
> `okf show runbooks/agent-bootstrap knowledge` — **read this first every session.**

**4e. Add `.agents/skills/okf-memory/` to `.gitattributes` as `linguist-generated`** so it doesn't pollute repo language stats (optional, cosmetic).

Commit as `feat(skills): integrate OKF search/validate into orient, plan, archive`.

---

### Phase 5 — Expose OKF as MCP (optional, only if Phases 1–4 land) · ~10 min

Add to `.mcp.json`:
```json
"okf": {
  "command": "okf",
  "args": ["mcp", "/Users/yebyen/w/mecris/knowledge"]
}
```
Test: restart pi, confirm `okf_search`/`okf_show` (or whatever names it exposes) appear. Update `runbooks/agent-bootstrap` step 3 to prefer the MCP tools over shelling out. If the MCP server doesn't work cleanly, revert and note it in `decisions/`.

---

## 3. Shipping

1. Current unpushed commit `748f8cbc` is on local `main`. Move it:
   ```
   git branch okf/knowledge-bootstrap
   git reset --hard origin/main
   git checkout okf/knowledge-bootstrap
   ```
2. Do Phases 1–4 (5 optional) as separate commits on that branch.
3. `git push -u origin okf/knowledge-bootstrap`, open PR titled **"OKF knowledge bundle: bootstrap + corrections + loop integration"**. Body: link this plan, paste final `okf validate` line, list deleted concepts and why.
4. Run `/mecris-pr-test`.
5. `/mecris-archive` — this is the first session where archive should itself exercise Phase 4c.

---

## 4. Definition of Done

- [ ] `okf validate knowledge --strict --drift` → 0/0/0/0
- [ ] Concept count **≤ 20** (fewer, better — we started at 18 with ~6 worth keeping)
- [ ] Every concept has `sources:` with ≥1 repo path and a non-generic `generated.by`
- [ ] Zero claims contradicted by `DEPLOYMENT.md`, `ARCHITECTURE.md`, `ls services/`
- [ ] `runbooks/agent-bootstrap` exists and a fresh agent can go from cold start → successful `mecris_get_narrator_context` by following it alone
- [ ] `/mecris-orient` SKILL.md references `okf`
- [ ] Pre-commit hook blocks an invalid `knowledge/` change (test it once)
- [ ] PR open, not pushed to `main`

## 5. Explicitly Out of Scope (resist)

- Auto-generating a concept per Python module / MCP tool (the 2,350-file trap)
- Vector DB / embeddings — BM25 over ≤20 docs is instant
- Documenting Android internals beyond auth lessons — Android has its own docs
- Migrating `docs/*.md` wholesale into OKF — OKF *points at* docs, it doesn't replace them

## 6. Success Metric (measure next session)

Count tool calls from session start → first *useful* action. This session: ~35 (including 20 wasted on auth discovery and skill-file re-reads). Target: **≤ 8** (`okf show bootstrap`, `mecris login`, `narrator_context`, `okf search <topic>`, then work).
