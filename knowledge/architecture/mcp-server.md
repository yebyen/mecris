---
type: Architecture
title: MCP Server (`mcp_server.py`)
description: The Python stdio MCP server is the interactive integration layer for accountability, budget, language, scheduling, GitHub, and health operations.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: mcp_server.py
  - resource: .mcp.json
  - resource: docs/MCP_INTEGRATION_SPEC.md
---

# MCP Server

`mcp_server.py` runs through `uv --project /Users/yebyen/w/mecris ... --stdio` as configured in `.mcp.json`. It exposes the operational surface used by agents: narrator context, Beeminder status, budget and budget-governor controls, language review controls, scheduler and notification operations, data portability, GitHub helpers, and system health.

Optional tool families are lazy-loaded with `mecris_load_tools`; use that rather than assuming every capability is active. Authentication is supplied by the Mecris login flow and persisted local credentials. The server should be treated as the source of live state; OKF stores durable interpretation and runbooks, not copies of volatile dashboards.

# Related Concepts
- [Budget Governor Service](services/budget-governor.md): MCP exposes budget-governor checks and records
- [Language Coaching Service](services/coaching-service.md): MCP exposes language coaching status and controls
- [PocketID Authentication and Session Lifecycle](services/auth-service.md): MCP access depends on authenticated Mecris sessions
- [Narrator Context as Primary Agent Sensor](narrator-context.md): Narrator context is an MCP operation
