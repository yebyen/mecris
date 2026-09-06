---
type: Architecture
title: Neon Data Architecture — The Forest Floor
description: Neon Postgres is the persistent truth layer for user-scoped goals, tokens, language state, usage, messages, autonomous turns, budgets, and scheduler elections.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: mecris-go-spin/schema.sql
  - resource: migrations
  - resource: docs/DATA_ARCHITECTURE_AND_PRIVACY.md
  - resource: mcp_server.py
---

# Neon Data Architecture

Neon Postgres is the durable state layer behind the local MCP and cloud sync paths. The user-scoped tables include `users`, `token_bank`, `walk_inferences`, `language_stats`, `goals`, `message_log`, `usage_sessions`, `autonomous_turns`, `budget_tracking`, and `scheduler_election`.

Every query must preserve the tenant boundary with `user_id` (ROADMAP Goal 0). The GDPR delete path removes `token_bank` first because it does not cascade, then deletes the user and dependent rows. Treat database schema and migrations as authoritative; this concept is a navigation summary, not a second schema.

# Related Concepts
- [Cloud Easing: Local Python MCP as Primary Backend](../../decisions/2026-06-cloud-easing.md): The local-first decision changes which persistence path is primary
