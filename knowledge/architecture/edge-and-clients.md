---
type: Architecture
title: Edge Runtimes & Clients
description: The misnamed mecris-go family: Android client, Rust/Spin edge components, and a small Go sync service.
generated: { by: agent/claude-fable-5.1, at: 2026-09-06T17:30:00Z }
sources:
  - resource: mecris-go/README.md
  - resource: mecris-go-project/README.md
  - resource: mecris-go-spin/DEPLOYMENT.md
  - resource: ARCHITECTURE.md
---

# Edge Runtimes & Clients

## Directory truth
- `mecris-go/` is a disambiguation README, not an implementation.
- `mecris-go-project/` is the Android/Kotlin Gradle app: PocketID/AppAuth authentication, Health Connect ingestion, and WorkManager workers.
- `mecris-go-spin/` contains Spin/WASM edge components, predominantly Rust (`*-rs`), plus the Go `sync-service`.

## Deployment truth
Akamai Functions (`*.fwf.app`) is the active authoritative cloud target with Neon Postgres and encrypted Twilio configuration. Fermyon Cloud is inactive and deprovisioned after WASM instantiation failures. The local Python MCP remains the primary interactive backend.

The “Go” name means Mecris on the go/mobile lineage; it does not mean the whole family is implemented in Go.

# Related Concepts
- [Neon Data Architecture — The Forest Floor](data/neon-db.md): Edge sync components and clients persist user-scoped state
- [Cloud Easing: Local Python MCP as Primary Backend](../decisions/2026-06-cloud-easing.md): Deployment posture governs which edge runtime is authoritative
