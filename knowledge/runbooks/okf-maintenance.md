---
type: Runbook
title: OKF Knowledge Base Maintenance
description: Periodic tasks to keep the OKF bundle healthy: validate, check for stale concepts, add missing sources, and prune low-value nodes.
generated: { by: agent/gpt-5.6-sol, at: 2026-09-06T23:20:00Z }
sources:
  - resource: AGENTS.md
  - resource: Makefile
  - resource: .githooks/pre-commit
  - resource: https://github.com/kingdonb/mecris/pull/296
---

# OKF Knowledge Base Maintenance

Run this runbook regularly (e.g., weekly or after a series of changes) to ensure the curated knowledge base remains accurate and useful.

## Maintenance Tasks

1. **Validate the bundle**
   ```bash
   make okf-validate
   ```
   Ensure zero errors and zero warnings.

2. **Check for stale concepts**
   ```bash
   make okf-check-drift
   ```
   The `okf validate --strict --drift` command reports a stale count; review any concepts marked stale and decide whether to update, archive, or remove them.

3. **Add missing sources**
   Search for concepts with empty or incomplete `sources:` lists and add appropriate references (commit URLs, file paths, or external docs).

4. **Prune low‑value nodes**
   If a concept’s body does not answer “What would I have to read/run to learn this without OKF?”, consider deleting it or merging its content into a higher‑level concept.

5. **Update stale_after dates**
   Adjust the `stale_after` frontmatter on time‑sensitive concepts (runbooks, decisions) as needed.

6. **Commit and validate**
   After making changes, run `make okf-validate` before committing.

## Automation
- The pre‑commit hook (`.githooks/pre-commit`) validates the knowledge bundle whenever a file under `knowledge/` is staged.
- The CI workflow (`.github/workflows/ci.yml`) installs the OKF CLI and runs validation on every push to a PR and on pushes to `main`.

## Maintenance evidence

After a substantial change, record why concepts changed—not just that files changed. The 2026-09-06 Pi status maintenance pass created a decision for deterministic status, corrected identity precedence, documented FastMCP's `structuredContent.result` wrapper, corrected narrator field semantics, and separated review-pump flow states from Beeminder worker priorities. This is the expected depth for a maintenance pass after architectural work.

## Related Concepts
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): The starting point for any agent session.
- [System Overview](../architecture/overview.md): The high‑level architecture that this knowledge base supports.
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Both runbooks address urgent situations: maintenance vs. emergency goal response.
- [Deterministic Pi Status and Progressive Context](../decisions/2026-09-06-deterministic-status.md): Example of the architectural knowledge a maintenance pass must preserve.
