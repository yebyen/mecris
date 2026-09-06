# OKF Documentation Assessment — 2026-09-06

## Method
Every doc in `docs/` rated 1-5 based on the OKF contract: "What would I have to read/run to learn this without OKF?"

| Score | Meaning | Action |
|---|---|---|
| 5 | High-value, real source, must reference in OKF | Onboard (add `sources:` link or create/update concept) |
| 3 | Real doc, medium re-read frequency, keep in repo | Reference in OKF sources, don't create separate concept |
| 1 | Low-value / superseded / historical only / empty | Move to `docs/attic/` or delete |

---

## 5/5 — Must Onboard / Reference

| Doc | Evidence | OKF Action |
|---|---|---|
| `BEEMINDER_ACTUAL_TRACKING.md` | Real tracking spec; feeds `beeminder_client.py` | Add to `architecture/beeminder-integration.md` sources |
| `BEEMINDER_LORE_CATALOG.md` | Catalog of all Beeminder concepts; used by `runbooks/beeminder-emergency.md` | Add to `runbooks/beeminder-emergency.md` sources |
| `BUDGET_GOVERNOR_SPEC.md` / `GUIDANCE.md` | Real Python service (`services/budget_governor.py`); 5%/5% envelope rule | Already in `architecture/services/budget-governor.md` sources; verify links |
| `NARRATOR_CONTEXT_ARCHITECTURE.md` | Defines `get_narrator_context()` JSON shape; feeds `architecture/narrator-context.md` | Add to `architecture/narrator-context.md` sources |
| `DATA_ARCHITECTURE_AND_PRIVACY.md` | Defines GDPR tables (`token_bank`, `walk_inferences`, etc.); feeds `architecture/data-model` concept | Add to `architecture/data-model.md` sources |
| `AUTH_CONFIGURATION.md` | PocketID v2.13, 1h access / 30-day refresh; feeds `services/auth-service.md` | Already referenced; verify |
| `DEPLOYMENT.md` (spin) | Akamai ACTIVE, Fermyon INACTIVE; feeds `architecture/edge-and-clients.md` | Add to `architecture/edge-and-clients.md` sources |
| `ROADMAP.md` | Strategic direction; feeds `philosophy.md` or new `architecture/roadmap.md` | Create `architecture/roadmap.md` concept |
| `SERVICE_GUIDE.md` | Service descriptions; feeds `services/*` concepts | Verify all 3 services have this in sources |
| `ARCHITECTURAL_EVOLUTION/01_the_bootstrap_era.md` | Historical design patterns (`Interceptor Pattern`, `Defensive Interceptor`, `Fallback Proxy`); valuable for architecture understanding | Add to `architecture/philosophy.md` or `knowledge/architecture/overview.md` sources |

---

## 3/5 — Keep in Repo, Reference Only (Don't Create New Concepts)

| Doc | Reason |
|---|---|
| `BEEMINDER_ASYNC_LORE.md` | Midnight Mandate + Reality Gap; already embedded in `runbooks/beeminder-emergency.md` |
| `BEEMINDER_PRIORITY_LORE.md` | 5-tier SNAPPY→UNDULY queue; already embedded in `runbooks/beeminder-emergency.md` |
| `MCP_INTEGRATION_SPEC.md` | MCP architecture spec; feeds `mcp-server.md` concept |
| `SETUP_GUIDE.md` / `QUICK_START.md` | Setup guides; superseded by `runbooks/agent-bootstrap.md` for agent use, but useful for human setup |
| `SECURITY_*.md` / `TRUST_BOUNDARY.md` / `BOOTSTRAP_KEY_MANAGEMENT.md` | Security reference; reference in `services/auth-service.md` |
| `TESTING_GUIDE.md` / `TESTING_CHECKLIST.md` | Test reference; reference in `.github/skills/mecris-pr-test/` if needed |
| `CLOUD_DEPLOYMENT_PLAN.md` / `DEPLOYMENT_PLAN_CDD.md` | Historical deployment docs; superseded by `architecture/edge-and-clients.md` and `decisions/2026-06-cloud-easing.md` |

---

## 1/5 — Move to `docs/attic/` or Delete

| Doc | Evidence | Action |
|---|---|---|
| `FULL_GUIDELINES.md` | 92 bytes — essentially empty | Delete |
| `CAVEMAN_SKILL.md` | Skill description; superseded by `.github/skills/` and `AGENTS.md` | Delete or move to attic |
| `GHOST_ARCHIVIST_SPEC.md` | Not in OKF, not active in current loop | Move to attic |
| `RELEASE_NOTES_v0.0.1-beta.4.md` | Old release; superseded by `session_log.md` and current bundle | Move to attic |
| `AGENT_AGENDA_DESIGN.md` / `AGENT_OPS_RUNBOOK.md` | Agent design docs; superseded by `.github/skills/` files + `runbooks/agent-bootstrap.md` | Move to attic |
| `THIRD_DEATH_STAR.md` | Metaphor/planning doc; not a capability-level concept | Move to attic |
| `TIMEZONE_SERVICE_FIX.md` | Specific fix; superseded by current time handling in `bin/mecris` / `cli/main.py` | Move to attic |
| `COMPREHENSIVE_MULTI_PROVIDER_BILLING.md` | Low re-read frequency; superseded by `budget-governor.md` concept | Move to attic |
| `archive/` (68 files) | Historical session logs and old plans; valuable for history but not for agent orientation | Create `docs/attic/archive/` and move all 68 files |

---

## Implementation Plan

1. Create `docs/attic/` and move all 1/5 docs (including `archive/` contents).
2. Verify `FULL_GUIDELINES.md` is deleted (it's empty).
3. Update OKF concept sources for 5/5 docs (add `docs/BEEMINDER_ACTUAL_TRACKING.md`, etc. to `sources:` lists).
4. Optionally create `architecture/roadmap.md` referencing `docs/ROADMAP.md`.
5. Validate: `make okf-validate` must still pass (0 errors, 0 broken links).
