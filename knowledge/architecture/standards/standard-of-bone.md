---
type: Architecture
title: JSON/WIT Communication Boundary
description: JSON and WIT define the portable communication boundary between Mecris services, clients, and edge components.
generated: { by: agent/cli, at: 2026-09-06T16:18:48Z }
sources:
  - resource: wit
  - resource: mecris-go-spin
  - resource: docs/MCP_INTEGRATION_SPEC.md
---

# Communication Boundary

JSON is the practical interchange format at MCP and API boundaries. WIT/component interfaces describe contracts for Spin/WASM components. Keep schemas and WIT files authoritative; use this concept to find them and to remember that the boundary exists to decouple local Python, Android, and edge runtimes.
