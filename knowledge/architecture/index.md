# Architecture
* [Mecris System Architecture Overview](overview.md) - Mecris is a personal accountability system centered on a Python MCP integration layer, user-scoped Neon state, Android/edge clients, and an auditable agent loop.
* [MCP Server (`mcp_server.py`)](mcp-server.md) - The Python stdio MCP server is the interactive integration layer for accountability, budget, language, scheduling, GitHub, and health operations.
* [Go Services (mecris-go, mecris-go-spin, mecris-go-project)](go-services.md) - Go-based services for performance-critical paths: mecris-go (core), mecris-go-spin (Spin/WASM cloud deployment), mecris-go-project (project tooling). Communicate via gRPC/HTTP with Python MCP server.
* [Beeminder Accountability Integration](beeminder-integration.md) - Beeminder is the live source for goal runways, derail risk, daily activity, and emergency prioritization.
* [Mecris Gall Loop](gall-loop.md) - The Mecris loop turns orientation into an auditable plan, bounded work, serialized state, and tested delivery.
* [Specialized Technical Skills Integration](specialized-skills.md) - Integrated skills from kingdon-skills repository: /author-skills, /follow-leader, /sdd, /atomic, /tdg, /ticket-author, /postmortem-author, /start-blasting, /hallucination-detector, /sos-emergency, plus infra/observability skills like /prometheus-status, /alertmanager-install, etc.
* [Mecris Architectural Philosophy: The Diseased Forest](philosophy.md) - Mecris uses a memorable forest/iron metaphor for a local-first accountability system whose durable truth is its user-scoped data.
* [Narrator Context as Primary Agent Sensor](narrator-context.md) - The narrator-context MCP response is the compact live situation report combining goals, budget, daily aggregate, system health, recommendations, and presence.
* [Daily Aggregate and Majesty Cake](daily-aggregate.md) - The daily accountability aggregate combines daily walk, Arabic review, and Greek review into a compact score consumed by the Majesty Cake widget.
* [Edge Runtimes & Clients](edge-and-clients.md) - The misnamed mecris-go family: Android client, Rust/Spin edge components, and a small Go sync service.
