---
type: Decision
title: Defer OKF MCP Server Integration
description: Keep OKF on the CLI for now: an stdio initialize smoke test succeeded but the server did not expose tools/list under the tested protocol, so .mcp.json remains unchanged.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: okf mcp knowledge smoke test, 2026-09-06
---

# Decision

An `okf mcp knowledge` stdio smoke test completed `initialize`, but a follow-up `tools/list` request returned `Method not found` for the tested protocol. Do not add an unverified `okf` server to `.mcp.json`.

Use the supported `okf show`, `okf search`, and `okf validate` CLI operations. Revisit MCP exposure only after confirming the supported protocol and tool/resource methods against the installed OKF release.

# Related Concepts
- [Agent Session Bootstrap](../runbooks/agent-bootstrap.md): Bootstrap continues to use the supported OKF CLI workflow
