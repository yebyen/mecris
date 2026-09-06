---
type: Architecture
title: Mecris Strategic Roadmap
generated: { by: agent/cli, at: 2026-09-06T21:40:25Z }
sources:
  - resource: docs/ROADMAP.md
  - resource: docs/ARCHITECTURE.md
  - resource: docs/OKF_IMPROVEMENT_PLAN.md
---

# Strategic Roadmap

The Mecris strategic direction (`docs/ROADMAP.md`, 2025-10-19) moves from the current local-development/accountability phase toward autonomous SMS-based operation.

## Current State → Target State

| Aspect | Current (Verified) | Target (Roadmap Phase 2-3) |
|---|---|---|
| **Interface** | Claude Code CLI, Pi harness (`.pi/extensions/mecris/`), local Python MCP (`mcp_server.py`) | SMS conversation (natural language, bidirectional) |
| **Operation** | Manual sessions with authentication (`bin/mecris login`) and lazy-loaded MCP families (`mecris_load_tools`) | Autonomous 24/7 operation (`system_pulse`, `presence` tracking) |
| **Deployment** | Local `.venv`, Akamai Functions (`ACTIVE`), local Python backend (`mcp_server.py`), PR-protected `main` branch | Production containers (`docs/CLOUD_DEPLOYMENT_PLAN.md`); Docker containerization planned (`#14`) |
| **User Experience** | Technical setup (`docs/SETUP_GUIDE.md`, `docs/QUICK_START.md`) | Pure SMS interaction (`docs/SETUP_GUIDE.md` legacy) |
| **Intelligence** | Human-guided (`mecris-orient`, `mecris-plan`, `mecris-archive`) with OKF knowledge bundle (21 concepts) | Intelligent automation (`BEEMINDER_ASYNC_LORE.md`: predictive insights, personalized coaching) |

## Key Milestones

### Phase 1: Production Foundation (Q4 2025 — Confirmed Complete in Bundle)
- [x] Documentation architecture (`docs/OKF_IMPROVEMENT_PLAN.md` completed; `docs/OKF_DOC_ASSESSMENT.md` rates 154 docs)
- [x] OKF bundle curated (21 concepts; `okf validate --strict --drift` clean)
- [x] Agent bootstrap (`runbooks/agent-bootstrap.md`) — cold-start procedure documented
- [x] Pre-commit hook (`.githooks/pre-commit`) and CI validation (`.github/workflows/ci.yml`)
- [x] Budget tracking (`services/budget_governor.py`) and Beeminder integration (`beeminder_client.py`)

### Phase 2: SMS Interface Development (Q1 2026 — Planned)
- Bidirectional SMS (`Twilio` configured; `docs/TWILIO_SETUP_GUIDE.md`)
- Conversation memory (`docs/BEEMINDER_ASYNC_LORE.md`: context preserved across message exchanges)
- Natural language responses (not robotic — see `docs/BEEMINDER_LORE_CATALOG.md`)
- Message queue with priority handling (`docs/BEEMINDER_PRIORITY_LORE.md`: SNAPPY/LOCKSY/BATCHY tier model applied to message processing)

### Phase 3: Advanced Intelligence (Q2 2026 — Planned)
- Predictive insights (`docs/NARRATOR_CONTEXT_ARCHITECTURE.md`: `recommendations` array improves over time)
- Personalized coaching (`docs/BEEMINDER_ACTUAL_TRACKING.md`: behavior patterns feed recommendations)
- Multi-source integration (`docs/MCP_INTEGRATION_SPEC.md`: calendar, email, weather, health data)
- Advanced heuristics without constant API usage (`docs/BEEMINDER_ASYNC_LORE.md`: Midnight Mandate reduces unnecessary sync calls)

## Dependencies & Risks

### External Dependencies (Verified Active or Configured)
- **Twilio SMS Service** (`docs/TWILIO_SETUP_GUIDE.md`): SMS delivery platform; configured in `.env`.
- **Akamai Functions** (`docs/DEPLOYMENT.md` implied; `mecris-go-spin/DEPLOYMENT.md`): `ACTIVE` (edge endpoints `*.fwf.app`); Fermyon `INACTIVE` (deprovisioned; see `decisions/2026-06-cloud-easing.md`).
- **Neon Postgres** (`docs/DATA_ARCHITECTURE_AND_PRIVACY.md`): Multi-tenant persistence (`user_id` boundary enforced; GDPR delete path removes `token_bank` first — no `CASCADE`).
- **Beeminder API** (`docs/BEEMINDER_LORE_CATALOG.md`): Live goal tracking; risk classification (`CRITICAL`/`WARNING`/`CAUTION`/`SAFE`).
- **Anthropic Admin API** (`docs/BUDGET_GOVERNOR_SPEC.md`): Budget tracking; 5%/5% envelope enforcement; `mecris_update_budget` MCP tool.
- **PocketID OAuth** (`docs/AUTH_CONFIGURATION.md`): Authentication; 1-hour access / 30-day sliding refresh tokens (`cli/main.py`: singleton `credentials_manager`).

### Risk Mitigation (Documented in OKF Decisions & Architecture)
- **Budget Constraints** (`decisions/2026-06-cloud-easing.md`, `decisions/2026-09-06-budget-extension.md`): Graceful degradation; period extended to `2026-10-07` (`31` days remaining); `budget_governor` gate triggers `WARNING`/`CRITICAL` recommendations.
- **Service Outages** (`architecture/edge-and-clients.md`): Local Python MCP (`mcp_server.py`) is primary backend; Akamai active; Fermyon deprovisioned.
- **Data Integrity** (`architecture/data-model.md` / `docs/DATA_ARCHITECTURE_AND_PRIVACY.md`): Multi-tenancy (`user_id` query boundary); no `CASCADE`; explicit serialization (`LOCKSY` tier for database writes).
- **Stale Data** (`runbooks/agent-bootstrap.md`, `runbooks/okf-maintenance.md`): `okf validate --strict --drift` runs on every commit; `stale_after` dates (`2026-12-05`) set on time-sensitive concepts.
- **Security** (`docs/AUTH_CONFIGURATION.md`, `.githooks/pre-commit`): PocketID v2.13; `.githooks/pre-commit` validates bundle; CI (`.github/workflows/ci.yml`) runs `okf validate`.

## Key Integration Points
- **Agent Bootstrap** (`runbooks/agent-bootstrap.md`): Cold-start procedure (`bin/mecris login` → `.venv` activation → `PYTHONPATH` → PocketID redirect → `user_id` UUID `c0a81a4b-115a-4eb6-bc2c-40908c58bf64`).
- **Gall Loop** (`architecture/gall-loop.md`): `orient` → `plan` → `work` → `archive` → `test`. Each skill (`.github/skills/`) references OKF (`runbooks/agent-bootstrap.md`, `runbooks/okf-maintenance.md`).
- **Knowledge Impact Tracking** (`.github/skills/mecris-plan/SKILL.md`, `AGENTS.md`): Spec issues include `"Knowledge impact"` line naming OKF concepts updated (`runbooks/agent-bootstrap.md`, `runbooks/okf-maintenance.md`).
- **Archive Validation** (`.github/skills/mecris-archive/SKILL.md`, `runbooks/okf-maintenance.md`): End-of-task checklist runs `make okf-validate`; `session_log.md` logs delta; `NEXT_SESSION.md` updates state.
- **Beeminder Emergency** (`runbooks/beeminder-emergency.md`, `docs/BEEMINDER_ASYNC_LORE.md`): `derail_risk` (`CRITICAL`/`WARNING`) triggers `SNAPPY`/`LOCKSY` priority; `safebuf == 0` triggers emergency procedure; Midnight Mandate (`midnight local time`) defines actual derailment window; smallest valid action (`submit to MCP tool`) satisfies user obligation.
- **Budget Extension** (`decisions/2026-09-06-budget-extension.md`, `docs/BUDGET_GOVERNOR_SPEC.md`): `mecris_load_tools("budget")` activates `budget_governor` MCP tools; `period_end` extended to `2026-10-07`; `days_remaining` `31`; `budget_health` `GOOD`.

## Metrics & Success Criteria
- **Bundle Health**: `okf validate --strict --drift` = `0` errors, `0` warnings, `0` broken links, `0` orphans, `0` stale (`21` concepts, `26` relationships).
- **Cold-Start Efficiency**: Target `≤ 8` tool calls from session start to first useful action (down from `~35` in first session; `runbooks/agent-bootstrap.md` saves `~20` discovery calls).
- **Maintenance**: Weekly `make okf-check-drift` + `okf validate`; `stale_after` dates set (`2026-12-05`) on `agent-bootstrap.md`, `beeminder-emergency.md`, `budget-extension.md`.
- **Automation**: `.githooks/pre-commit` validates `knowledge/` changes; CI (`.github/workflows/ci.yml`) validates on PR and `main` push; `Makefile` targets `okf-validate` and `okf-check-drift`.
- **Knowledge Growth**: Each session adds `≤ 1` durable concept; `runbooks/agent-bootstrap.md` is the starting point; no transient concepts created for daily goal completions.

---
*This roadmap is a living document. Updates recorded in `session_log.md` (e.g., `2026-09-06 — OKF knowledge base improvements and validation`) and `knowledge/decisions/` (`2026-09-06-cloud-easing.md`, `2026-09-06-okf-mcp-deferred.md`). For the full improvement plan that guided this bundle, see `docs/OKF_IMPROVEMENT_PLAN.md`. For the curated knowledge index, see `knowledge/index.md` and `runbooks/index.md`.*

## Related Concepts
- [Mecris System Architecture Overview](overview.md): The high-level architecture that this roadmap advances.
- [Edge Runtimes and Clients](edge-and-clients.md): Milestones include mobile (Android/Kotlin), Spin/WASM, and cloud deployment (Akamai active, Fermyon deprovisioned).
- [Philosophy](philosophy.md): Strategic direction and design principles underpinning the roadmap milestones.
- [Agent Memory Maintenance](../runbooks/okf-maintenance.md): The roadmap is maintained through the OKF maintenance process.
- [Mecris System Architecture Overview](overview.md): Roadmap advances the system architecture toward SMS vision and autonomous operation.
- [Mecris Architectural Philosophy: The Diseased Forest](philosophy.md): Strategic milestones align with design principles.
- [Edge Runtimes & Clients](edge-and-clients.md): Milestones include mobile (Android), Spin/WASM, and Akamai cloud deployment.
- [OKF Knowledge Base Maintenance](../runbooks/okf-maintenance.md): Roadmap is maintained through OKF validation and maintenance tasks.
