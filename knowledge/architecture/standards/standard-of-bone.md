---
type: Architecture
title: JSON/WIT Communication Boundary
description: JSON and WIT define the portable communication boundary between Mecris services, clients, and edge components.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: wit
  - resource: mecris-go-spin
  - resource: docs/MCP_INTEGRATION_SPEC.md
---

# Communication Boundary

JSON is the practical interchange format at MCP and API boundaries. WIT/component interfaces describe contracts for Spin/WASM components. Keep schemas and WIT files authoritative; use this concept to find them and to remember that the boundary exists to decouple local Python, Android, and edge runtimes.
