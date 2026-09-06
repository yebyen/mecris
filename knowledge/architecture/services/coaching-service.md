---
type: Architecture
title: Language Coaching Service
description: Python language-coaching operations synchronize review pumps and expose velocity/completion information used by daily accountability.
generated: { by: agent/cli, at: 2026-09-06T16:18:48Z }
sources:
  - resource: services/coaching_service.py
  - resource: docs/linguistics
  - resource: mcp_server.py
---

# Language Coaching Service

`services/coaching_service.py` supports language-learning velocity, review-pump levers, and synchronization triggers. The daily aggregate consumes Arabic and Greek review state; it is not a generic “Go service.”

Use live MCP status before changing a lever. Persist durable decisions or unusual failure modes in OKF, while keeping routine card counts in the source systems. The coaching service is related to Beeminder because language work is both a learning workflow and an accountability commitment.

# Related Concepts
- [Daily Aggregate and Majesty Cake](../daily-aggregate.md): Language review state contributes to the daily aggregate
