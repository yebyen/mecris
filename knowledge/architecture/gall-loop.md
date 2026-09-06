---
type: Architecture
title: Mecris Gall Loop
description: The Mecris loop turns orientation into an auditable plan, bounded work, serialized state, and tested delivery.
generated: { by: agent/gpt-5.6-sol, at: 2026-09-06T23:20:00Z }
sources:
  - resource: .github/skills/mecris-orient/SKILL.md
  - resource: .github/skills/mecris-plan/SKILL.md
  - resource: .github/skills/mecris-archive/SKILL.md
  - resource: .github/skills/mecris-pr-test/SKILL.md
  - resource: AGENTS.md
---

# Mecris Gall Loop

`/status` is not a Gall arm. It is a deterministic, model-free Pi command for a quick live check. Use `/mecris-orient` when project history, issues, commits, or OKF context are needed.

- **Orient / on-peek** (`.github/skills/mecris-orient`): reads `NEXT_SESSION.md`, recent commits, upstream sync, and open issues; produces a read-only situation report.
- **Plan / on-poke** (`mecris-plan`): opens one GitHub issue with Intent, Because, and Validation before work.
- **Work**: make the smallest change that satisfies the plan and update OKF when durable knowledge changes.
- **Archive / on-save** (`mecris-archive`): closes the plan issue, rewrites `NEXT_SESSION.md`, appends `session_log.md`, validates OKF, and commits.
- **PR-test / on-agent** (`mecris-pr-test`): dispatches and polls the complete pipeline for a PR.

Related skills include `mecris-bonsai`, `mecris-rebase`, `parity-arbitration`, `release-workflow`, `beeminder-audit`, and the chore inventory skills. They are actions around the loop, not separate memory stores.

# Related Concepts
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): Bootstrap is the cold-start prelude to the Gall loop
- [Critical Situation: Budget Expired, Walk Needed, Reviewstack Derailing Tomorrow](../decisions/2026-09-06-critical-situation.md): The historical snapshot was discovered through orientation
