---
type: Architecture
title: Mecris System Architecture Overview
description: Mecris is a personal accountability system centered on a Python MCP integration layer, user-scoped Neon state, Android/edge clients, and an auditable agent loop.
generated: { by: agent/cli, at: 2026-09-06T16:18:48Z }
sources:
  - resource: ARCHITECTURE.md
  - resource: README.md
  - resource: .mcp.json
  - resource: AGENTS.md
---

# Mecris System Architecture

Mecris combines a local Python MCP server, a Neon Postgres persistence layer, an Android/Kotlin client, and Akamai/Spin edge components. The MCP server is the agent-facing integration layer; live operational state stays in Mecris and Beeminder, while OKF stores curated architecture, decisions, and runbooks.

The development process is the Gall loop: orient, plan, work, archive, and test. The system is deliberately local-first for interactive agent work, with cloud deployment available for edge/mobile synchronization. Read the linked concepts rather than loading the entire repository for routine planning.

# Related Concepts
- [MCP Server (`mcp_server.py`)](mcp-server.md): Python stdio MCP is the agent-facing integration layer
- [Edge Runtimes & Clients](edge-and-clients.md): Android and edge runtimes extend the local MCP system
- [Beeminder Accountability Integration](beeminder-integration.md): Beeminder supplies live accountability state
- [Mecris Gall Loop](gall-loop.md): The Gall loop governs agent work and state serialization
- [Mecris Architectural Philosophy: The Diseased Forest](philosophy.md): The metaphor records system intent and local-first posture
- [Neon Data Architecture — The Forest Floor](data/neon-db.md): Neon Postgres is the user-scoped durable state layer
- [Narrator Context as Primary Agent Sensor](narrator-context.md): Narrator context is the compact live sensor used during orientation
- [Daily Aggregate and Majesty Cake](daily-aggregate.md): Daily aggregate summarizes today's accountability components
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): Every agent session begins with the bootstrap runbook
- [Beeminder Emergency Response](../runbooks/beeminder-emergency.md): Urgent goal risk is handled by the Beeminder emergency runbook
- [JSON/WIT Communication Boundary](standards/standard-of-bone.md): JSON and WIT define portable component communication boundaries
