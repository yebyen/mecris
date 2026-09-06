---
type: Decision
title: Cloud Easing: Local Python MCP as Primary Backend
description: After Akamai/Fermyon deployment failures, local Python MCP became the primary interactive backend; the deployment pivot remains reversible.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
stale_after: 2026-12-05
sources:
  - resource: ARCHITECTURE.md
  - resource: mecris-go-spin/DEPLOYMENT.md
---

# Decision

## Current decision
Use the local Python MCP as the primary backend for interactive agent sessions. Keep Akamai as the active authoritative edge deployment for cloud/mobile paths. Fermyon is inactive and deprovisioned after WASM instantiation failures.

## Rationale and reversibility
The exact cloud failure mechanism was not fully isolated, so this is a tactical reliability decision, not a permanent rejection of cloud deployment. Revisit it by bisecting deployment history and reproducing the runtime capability failure before changing the primary path.
