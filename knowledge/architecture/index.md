# Architecture
* [Mecris System Architecture Overview](overview.md) - Mecris is a personal accountability system centered on a Python MCP integration layer, user-scoped Neon state, Android/edge clients, and an auditable agent loop.
* [MCP Server (`mcp_server.py`)](mcp-server.md) - The Python stdio MCP server is the live integration layer for accountability, budget, language, scheduling, and health operations.
* [Beeminder Accountability Integration](beeminder-integration.md) - Beeminder is the live source for goal runways, derail risk, daily activity, and emergency prioritization.
* [Mecris Gall Loop](gall-loop.md) - The Mecris loop turns orientation into an auditable plan, bounded work, serialized state, and tested delivery.
* [Mecris Architectural Philosophy: The Diseased Forest](philosophy.md) - Mecris uses a memorable forest/iron metaphor for a local-first accountability system whose durable truth is its user-scoped data.
* [Narrator Context as Primary Agent Sensor](narrator-context.md) - The narrator-context MCP response is the compact live situation report combining goals, budget, daily aggregate, system health, recommendations, and presence.
* [Daily Aggregate and Majesty Cake](daily-aggregate.md) - The daily accountability aggregate combines daily walk, Arabic review, and Greek review into a compact score consumed by the Majesty Cake widget.
* [Edge Runtimes & Clients](edge-and-clients.md) - The misnamed mecris-go family: Android client, Rust/Spin edge components, and a small Go sync service.
* [Strategic Roadmap](roadmap.md) - Project milestones (Phase 1-3), SMS vision, and dependency tracking.
* [Beeminder Majesty Cake Integration](beeminder-majesty-cake.md) - The Majesty Cake widget visualizes daily accountability as X/Y goals satisfied, drawing from Beeminder goal data and the daily aggregate of walk, Arabic review, and Greek review.
