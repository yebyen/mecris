---
type: Architecture
title: PocketID Authentication and Session Lifecycle
description: PocketID/AppAuth authentication uses shared repository state, rotating refresh tokens, and bounded refresh operations across Android and background workers.
generated: { by: agent/cli, at: 2026-09-06T16:18:48Z }
sources:
  - resource: services/auth_service.py
  - resource: services/auth_server.py
  - resource: session_log.md
  - resource: docs/AUTH_CONFIGURATION.md
---

# Authentication Service

The Python service handles Mecris API authentication, while the Android client uses PocketID/AppAuth. The documented production lesson is that PocketID rotates refresh tokens as single-use credentials: `MainActivity`, view models, and WorkManager workers must share the singleton `PocketIdAuthRepository` rather than replay independent in-memory tokens.

The current hardening includes a one-hour access-token lifetime, a sliding refresh window, explicit refresh-token persistence, fresh error state on re-auth, and timeouts around token exchange. Do not dispose the shared authorization service from an activity lifecycle. When changing auth, read the session-log incident and `docs/AUTH_CONFIGURATION.md` first.

# Related Concepts
- [Agent Session Bootstrap](../../runbooks/agent-bootstrap.md): Authentication is required before live MCP calls
