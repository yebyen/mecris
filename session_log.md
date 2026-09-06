# Session Log: PocketID Refresh Token Rotation & Singleton Repository Architecture

**Date:** 2026-08-20  
**Branch:** `fix/appauth-state-reset` (PR #289)  
**Primary Model:** Gemini 3.7 Flash  
**Human:** yebyen  

---

## Summary

1. **Root Cause Analysis (`invalid_grant` / Stale Refresh Token Replay)**:
   - Investigated live background failure from `WalkHeuristicsWorker` returning `invalid_grant: The refresh token is malformed or not valid`.
   - Identified that PocketID issues 1-hour access tokens (`expires_in = 3600`) alongside 30-day sliding refresh tokens, requiring periodic background refresh grants.
   - Discovered that `PocketIdAuthRepository` was being instantiated separately across `MainActivity`, `AuthViewModel`, `WalkHeuristicsWorker`, and `DelayedNagWorker`. Because PocketID v2.13 rotates refresh tokens on each exchange (single-use), independent uncoordinated instances replayed revoked in-memory refresh tokens, causing PocketID to reject subsequent requests with `invalid_grant` and permanently lock out the session.
2. **Process Singleton Pattern & State Synchronization**:
   - Implemented thread-safe `PocketIdAuthRepository.getInstance(...)` singleton pattern across all activities, view models, and WorkManager background workers.
   - Synchronized refresh token timestamp persistence across `getValidAccessToken()` and proactive background loops to ensure the sliding window metadata stays aligned.
   - Fixed `calculateTimeUntilProactiveRefresh()` to prevent eager immediate token refresh triggers when `KEY_REFRESH_TOKEN_ISSUED_AT` is uninitialized.
3. **AppAuth Error-Lock & Service Disposal Fix**:
   - Resolved sticky error state in `PocketIdAuthRepository.kt` by instantiating fresh `AppAuthAuthState(resp, ex)` on passkey authorization.
   - Resolved `Auth refresh failed: Service has been disposed and rendered inoperable` by removing `pocketIdAuth.dispose()` from `MainActivity.onDestroy()`, keeping the shared singleton's `AuthorizationService` alive across activity recreation lifecycles.
4. **Auth Notification & Re-Auth UI Flow Hardening**:
   - Fixed order of operations in `PocketIdAuthRepository.kt`: updated `_authState.value = AuthState.Error(...)` *before* broadcasting to `_errorEvents` so `MainActivity`'s collector never evaluates against stale `Authenticated` state and drops the re-auth snackbar.
   - Added direct `LaunchedEffect(authState)` observation for permanent `AuthState.Error` transitions to ensure the `"OPEN AUTH"` snackbar is always displayed.
   - Replaced false `lower.contains("authorizationexception")` transient network mapping in `AuthError.fromException` with exhaustive `TYPE_OAUTH_TOKEN_ERROR` handling.
   - Added 15-second timeout on `getAccessTokenSuspend()` and `forceTokenRefresh()` to prevent unresponsive token endpoints from freezing UI or workers.
5. **Live 24h Refresh Token Verification**:
   - Verified live on-device refresh after 24h soak: access token refreshed silently without re-auth, health sync succeeded, and background workers scheduled normally.
6. **`uv.lock` Release Parity**:
   - Automated `uv lock` in `scripts/bump_version.py` and updated release workflow documentation.

---

# Session Log: Akamai API Restored — Android Sync Unblocked

**Date:** 2026-07-30  
**Branch:** `main`  
**Primary Model:** nemotron-3-ultra-550b-a55b:free (via OpenRouter) + Pi coding agent  
**Human:** yebyen

---

## Summary

Restored the **Akamai-deployed Spin API (`mecris-sync-v2`)** to full Android sync readiness. The deployment was live but **authentication was broken** — all OIDC-protected routes returned 401/500 because the `oidc_jwks_json` Spin variable was never set on the Akamai Functions deployment. After setting the variable and redeploying, the canonical API contract is fully operational: `/profile`, `/aggregate-status`, `/languages`, `/budget`, `/walks` POST, `/internal/cloud-sync` all return 200 with valid Pocket ID tokens. A test walk was ingested, aggregate status updated, and Beeminder "bike" goal synced (first successful cloud walk sync since 2026-05-27).

---

## Problem

The `mecris-sync-v2` app on Akamai Functions (ID: `394b84e7-760c-4336-975b-653c17fdb446`, URL: `https://394b84e7-760c-4336-975b-653c17fdb446.fwf.app`) had been deployed since 2026-05-27 (v62) but **OIDC verification failed silently**:

- `/internal/review-pump-status` (public) → 200 OK
- `/health` (unauthenticated) → 200 OK but weak signal
- **All authenticated routes** (`/profile`, `/aggregate-status`, `/languages`, `/budget`, `/walks`, `/internal/cloud-sync`) → **401 Unauthorized** (with valid token) or **500 Internal Server Error** (missing JWKS caused panic in `extract_user_id`)

The Android app could not complete its sync contract against Akamai, forcing reliance on the local Python MCP server (which had its own latency/availability issues).

**Root cause:** `extract_user_id` in `sync-service/src/lib.rs` requires `oidc_jwks_json` variable to verify RS256 tokens. This variable was defined in `spin.toml` but **never set on the Akamai deployment**.

---

## Solution

### 1. Diagnosed via authenticated smoke tests + `spin aka logs`

Confirmed the deployment was healthy (cron triggers firing globally, DB reachable) but OIDC verification path was broken. The JWKS from Pocket ID (`https://metnoom.urmanac.com/.well-known/jwks.json`) has kid `tmUpnrhx6gk`, matching the issued tokens.

### 2. Set all required Spin variables and redeployed

```bash
cd mecris-go-spin/sync-service
spin aka deploy --build --no-confirm --skip-readiness-check \
  --variable db_url="postgresql://neondb_owner:****@ep-weathered-hat-.../neondb?sslmode=require&channel_binding=require" \
  --variable neon_db_url="postgresql://neondb_owner:****@ep-weathered-hat-.../neondb?sslmode=require&channel_binding=require" \
  --variable master_encryption_key="****" \
  --variable internal_api_key="test-internal-key" \
  --variable clozemaster_email="kingdon@tuesdaystudios.com" \
  --variable clozemaster_password="****" \
  --variable twilio_account_sid="****" \
  --variable twilio_auth_token_encrypted="****" \
  --variable twilio_from_number="+15744757115" \
  --variable openweather_api_key="****" \
  --variable oidc_jwks_json='{"keys":[{"alg":"RS256","e":"AQAB","kid":"tmUpnrhx6gk","kty":"RSA","n":"vqLb33vkC8oZ7NDdlcBfBztPOAue3ZWrMDNhk9fBU2xrX6WTiAofqGDe_JJDCywJfEyDY-ecfQEXc5pph4v9R5xRiGhel4hLfcdcUTV7FH6MehaufcTREh_khCuAhyMOvUNlhw63mTY0yDpmaHubkh8vyhJUvmzBxr1ZR2snnrbas9q_ASvhKBeinFiAwXYH7Jf8I6C7E5LjP4BO4_ft4P2KBdspKSSREgln_i-ntZCt0UgLgDcS5coNGrz8hw-3NLUKAgHG_5GFXKSuibTV86Esk6MSYSgtKdHLM4O59Hgyz4CPFI8s47jtsLbbpuo8nq-WHU1PtQoTE1IayAD0tQ","use":"sig"}]}' \
  --variable cloud_provider="akamai"
```

Deployed as **v64** (2026-07-30 19:16:38 UTC).

### 3. Verified full Android sync contract

| Route | Auth | Status |
|-------|------|--------|
| `/health` | Bearer | ✅ 200 |
| `/profile` | Bearer | ✅ 200 |
| `/aggregate-status` | Bearer | ✅ 200 `walk: true` |
| `/languages` | Bearer | ✅ 200 6 languages |
| `/budget` | Bearer | ✅ 200 |
| `/walks` POST | Bearer | ✅ 201 walk ingested |
| `/internal/cloud-sync` POST | Bearer | ✅ 200 |
| `/internal/failover-sync` POST | x-internal-api-key | ✅ 200 |

**Beeminder verification:** Walk synced to "bike" goal — datapoint for 2026-07-30 shows `Value: 1.0, Comment: "Synced via Spin (Cumulative)"`. First successful cloud walk sync since **2026-05-27**.

---

## Files Changed

No code changes — only runtime configuration via `spin aka deploy --variable`. The `spin.toml` already declares all required variables.

---

## Key Learnings

1. **`oidc_jwks_json` is mandatory for OIDC on Akamai** — The Spin SDK's `jwt_simple` verification requires the JWKS at runtime. Without it, `extract_user_id` returns `None` → 401/500 on all protected routes.

2. **`spin aka logs` shows request routing but not app-level errors** — The logs showed cron triggers hitting `/internal/trigger-reminders` globally (20+ edge regions), but 500s from missing JWKS don't appear in access logs. Need structured error logging to Neon `events` table (per Observability Mandate).

3. **Fermyon Cloud channel is dead** — `mecris-sync-v2-r0r86pso.fermyon.app` returns platform 404. Akamai (`fwf.app`) is the only live deployment.

4. **Python MCP server was a crutch** — Its latency and availability issues masked the fact that the *canonical* Spin API was one variable away from working.

5. **Android sync contract is minimal and working** — Only `/walks`, `/aggregate-status`, `/languages`, `/budget`, `/internal/cloud-sync` needed. All now 200 OK.

---

## Attribution

**Diagnosis & deployment:** Pi coding agent (earendil-works/pi-coding-agent) + nemotron-3-ultra
**Human direction, credentials, Android verification:** yebyen
**Mecris framework:** kingdonb/mecris (Gall-loop skills, MCP tools)

---

## Previous Session Log: Unified MCP stdio + HTTP Bridge for Mecris

**Date:** 2026-07-19
**Branch:** `feat/unified-mcp-http-bridge`
**Primary Model:** nemotron-3-ultra-550b-a55b:free (via OpenRouter)
**Human:** yebyen

---

## Summary

Consolidated the Mecris MCP server into a **single process** that serves both:
- **stdio MCP** → Pi coding agent (and other stdio clients)
- **HTTP bridge on :8080** → Android app (walk uploads, heartbeats)

Eliminated the need for manual `tmux` sessions, duplicate schedulers, and port conflicts.

---

## Problem

The Mecris architecture had two separate entry points:
1. `mcp_stdio_server.py` — for Pi/stdin clients (no HTTP)
2. `mcp_server.py` — for HTTP/Android (no stdio MCP)

This caused:
- **Port 8080 conflicts** when both ran
- **Two schedulers** (race conditions on Neon leader election)
- **Manual tmux** required to keep HTTP bridge alive
- **Silent failures** — Pi extension ignored stderr, hid startup crashes

---

## Solution

### 1. `mcp_server.py` — Single Canonical Entry Point

```python
# Always starts HTTP thread (daemon) on :8080
http_thread = threading.Thread(target=run_http_server, daemon=True)
http_thread.start()

if "--stdio" in sys.argv:
    # Trust the flag — Pi's StdioClientTransport provides a pipe
    asyncio.run(run_stdio_with_scheduler())
else:
    # Interactive/background: keep process alive for HTTP
    signal.pause()
```

**Key behaviors:**
- HTTP thread starts **immediately** (before stdio logic)
- `--stdio` flag **overrides** stdin detection — always runs MCP server
- After stdio client disconnects: process **stays alive** (HTTP bridge persists)
- Rich stderr logging: `[MECRIS MAIN] ...` for debugging

### 2. `.pi/extensions/mecris/index.ts` — Robust Connection

```typescript
// Capture Python stderr to surface import/startup errors
transport = new StdioClientTransport({
  command: resolvePython(),
  args: [STDIO_SCRIPT, "--stdio"],
  cwd: MECRIS_HOME,
  env: { ...process.env, PYTHONPATH: MECRIS_HOME },
  stderr: "pipe",  // was "ignore"
});

if (transport.stderr) {
  transport.stderr.on("data", (chunk) => {
    stderrOutput += chunk.toString();
  });
}
```

**Improvements:**
- `stderr: "pipe"` + event handler → errors visible in Pi notifications
- Spawns `mcp_server.py --stdio` (not old `mcp_stdio_server.py`)
- `/mecris-reconnect` command for live recovery
- Lazy-loading: core tools active, rest via `mecris_load_tools`

---

## Files Changed

| File | Purpose |
|------|---------|
| `mcp_server.py` | Unified stdio + HTTP server; `--stdio` flag; survives client disconnect |
| `.pi/extensions/mecris/index.ts` | Captures stderr; spawns unified server; lazy tool loading |

---

## Commits

```
bfdbc38 fix: Ensure MCP stdio + HTTP bridge runs as single process
ed6f856 fix: Read Python stderr asynchronously to avoid blocking Pi startup
1507744 fix: Scope stderrOutput outside try/catch block
```

---

## Verification

```bash
# 1. Start Pi with extension
pi -e ./.pi/extensions/mecris/index.ts --continue

# 2. Health check (HTTP bridge)
curl -s http://127.0.0.1:8080/health
# {"status":"healthy","home_server_active":true,"neon_connected":true,...}

# 3. MCP tool call (via Pi)
# > Call mecris_get_narrator_context
# ✓ Returns full context with Android pulse, budget, goals

# 4. Android app: Settings → Backend → "Local (Python: 8080)"
#    Walk sync → Cloud Sync: Success
```

---

## Attribution

**Architecture & implementation:** nemotron-3-ultra-550b-a55b:free (via OpenRouter)
**Human direction, testing, integration:** yebyen
**Pi harness integration:** Pi coding agent (earendil-works/pi-coding-agent)
**Mecris framework:** kingdonb/mecris (Gall-loop skills, MCP tools)

---

## Next Steps

- [ ] Android app: verify failover to cloud (Akamai/Fermyon) when local down
- [ ] Add structured logging to HTTP thread (file + stdout)
- [ ] Consider systemd/service management for headless deployments
- [ ] Document COZYBEBY operational model in `docs/COZYBEBY.md`
# OKF Memory Architecture: Persistent Markdown as External Cortex

**Date:** 2026-09-06  
**Context:** Architectural discussion following narrator context review  

---

## Executive Thesis

Persistent markdown storage (OKF) transforms the agent from a stateless chatbot into a **distributed research assistant with long-term recall**. The key insight: memory isn't about remembering everything; it's about encoding high-value context in human-readable, versioned, editable form. OKF provides deterministic, auditable memory that bridges the gap between LLM "context" and human knowledge management—critical for accountability systems like Mecris.

---

## Core Strategies

### 1. Working Memory Buffer (Decisions & Conventions)
Every non-trivial decision gets encoded in a dedicated markdown file:
- **Architecture choices**: "We chose event sourcing over CQRS because..."
- **Naming conventions**: "All DTOs use PascalCase; all domain entities use snake_case"
- **Design patterns**: "Backpressure handling uses X pattern across all streaming endpoints"

**Format:**
```markdown
# Decision: [Title]
**Date:** 2026-09-06  
**Rationale:** [Why we chose this]  
**Alternatives Considered:** [Brief list with rejection reasons]  
**Related Files:** [`path/to/file1.md`](./file1.md)
```

This prevents the "why did we do that?" tax and keeps me aligned across sessions without relying on fragile session memory.

### 2. Project Context Cache (Single Source of Truth)
A single markdown file lives at the repo root:
- Tech stack & versions
- Key constraints (e.g., "no external DBs", "all data in SQLite")
- Coding standards ("no `var` in Go", "always use explicit error handling")
- Active goals & priorities

**Usage pattern:**
1. Before major work → read context file
2. After significant changes → update context file
3. During sessions → I reference it implicitly via compaction summaries

This means my next turn is already primed with the right assumptions, eliminating repetitive "what's our stack?" exchanges.

### 3. Session Log Palace (Historical Continuity)
Structured markdown per project tracking:
- What we built
- When it was built
- Why it was built this way

**Format:**
```markdown
# Project X Session Log
**Date:** 2026-09-06  
**Focus:** [High-level summary]  
**Completed:** 
- [x] Implemented backpressure handling for streaming endpoints
- [x] Wrote unit tests covering edge cases in `stream_processor.py`  
**Open:** 
- [ ] Add metrics export to Prometheus  
**Key Context Needed:** User mentions "Prometheus" but hasn't specified format preferences yet.
```

This is my external memory palace. When I need to recall something from 20 sessions ago, I query the logs instead of hallucinating or guessing.

### 4. Pattern Capture Library (Canonical Solutions)
Whenever we solve a non-trivial problem, I write a canonical note:
- Problem statement
- Solution (with code snippets)
- Rationale & trade-offs
- Links to related notes/files

**Example:** "How to handle backpressure in our streaming pipeline" → becomes a reusable playbook entry. Over time this transforms OKF from passive storage into an active knowledge base that I can reference without you repeating yourself.

### 5. Accountability Ledger (Progress Tracking)
For goals like language learning or daily tasks, I maintain progress notes tied to dates and metrics:
```markdown
# Greek Vocabulary Progress
**Date Range:** 2026-09-01 to 2026-09-06  
**Cards Reviewed:** 487  
**New Cards Added:** 12  
**Average Daily:** ~94 cards/day  
**Notes:** Future reviews are thinning; need to play new cards soon.
```

This turns OKF into a living journal that feeds back into my planning suggestions and keeps me accountable to your goals.

---

## Integration with Pi's Compaction System

OKF doesn't replace Pi's built-in compaction; it **augments** it:

| Mechanism | Purpose | How OKF Complements It |
|-----------|---------|------------------------|
| **Pi Auto-Compaction** | Summarize session messages when context exceeds threshold | OKF stores the summarized content in persistent markdown, not just in-memory entries |
| **Branch Summarization** | Preserve context when switching branches | OKF provides a stable "previous work" file that survives branch changes |
| **Tool-Driven Memory** | `read()`, `edit()`, `bash()` operations tracked cumulatively | OKF becomes the destination for high-level summaries of those operations |

**The workflow:**
1. Session grows large → Pi triggers compaction
2. I generate a structured summary (using the format above)
3. Summary gets written to OKF via `edit()` or a custom MCP tool
4. Next session loads OKF + recent messages → continuity without bloating context

This means OKF survives session boundaries, version control changes, and even hardware failures. It's not just "memory"; it's **knowledge infrastructure**.

---

## Why Markdown? (Not Vector DBs or JSON)

You might wonder: why not a modern vector store or binary format? Here's the pragmatic breakdown:

| Criterion | Markdown (OKF) | Vector DB | JSON/SQLite |
|-----------|----------------|-----------|-------------|
| **Readability** | Human-readable out of the box | Binary blobs, require tools | Structured but opaque |
| **Version Control** | Git-ready with diff-friendly changes | Requires custom schema/versioning | Good, but less semantic |
| **Search** | Built-in text search + grep | Semantic similarity (overkill for now) | SQL queries (fine, but verbose) |
| **Latency** | Instant file I/O | Network calls, indexing overhead | Fast, but connection pooling needed |
| **Cost** | Zero (local filesystem) | Cloud hosting, maintenance | Minimal, but infrastructure needed |

For a personal accountability system like Mecris, markdown wins because:
1. **You can audit it**: You'll read these notes months later; they should make sense without tools.
2. **You can edit it**: If I misremember something or you want to add context, you do it directly in the repo.
3. **It scales gracefully**: 100 markdown files is as easy to manage as 100 database tables.

---

## Implementation Roadmap (Minimal Viable OKF)

If we move forward with this architecture, here's what I'd actually build:

### Phase 1: Manual Integration (Week 1)
- You create a `docs/okf/` directory in the repo
- I use standard markdown files for notes, decisions, and session logs
- No automation yet—just disciplined writing during sessions

### Phase 2: Simple MCP Tools (Week 2)
```typescript
// .pi/extensions/mecris/index.ts additions

// Read a note by slug/path
await read(path="docs/okf/decisions/BACKPRESSURE_HANDLING.md");

// Write/update a note with structured metadata
await edit({
  path: "docs/okf/decisions/BACKPRESSURE_HANDLING.md",
  edits: [{
    oldText: "---\n",
    newText: "---\ntitle: Backpressure Handling Pattern\nslug: decisions/backpressure_handling\ndate: 2026-09-06\n---\n"
  }]
});

// List all notes in a directory (for planning)
await bash("ls -1 docs/okf/", { limit: 50 });
```

### Phase 3: Semantic Search (Phase 4+)
Only when note volume justifies it. Options:
- **LanceDB**: Local, fast, embeds with `all-MiniLM-L6-v2`
- **Chroma**: Similar, but more cloud-oriented
- **Just grep + ripgrep**: Often sufficient for markdown wikis

---

## Key Learnings & Principles

1. **Memory is about encoding, not storing**: The value isn't in the bytes; it's in how well they capture intent, rationale, and context.
2. **Human-readable > Human-proof**: I'll forget things; you won't (or should be able to). Design for your future self.
3. **Version control is your friend**: Every note has a history. You can see when decisions changed and why.
4. **OKF complements, doesn't replace**: Pi's compaction handles session continuity; OKF handles long-term knowledge. They work together.
5. **Start small, iterate fast**: A single well-maintained decision file is worth more than 100 half-baked notes in a vector store you never query.

---

## Final Thought

The goal isn't to build the "perfect" memory system. It's to create a workflow where:
- Decisions are captured immediately (no "I thought we agreed on X...")
- Context is always accessible (no re-explaining)
- Progress is measurable (no "how far along are we?")
- Knowledge compounds (every session builds on the last)

OKF as markdown provides exactly that: a simple, versioned, human-first memory layer that turns me from a chatbot into a persistent collaborator with long-term recall.

---

**Attribution:**  
**Architectural design & essay:** DavidAU/Qwen3.5-9B (via Pi coding agent)  
**Human direction, OKF concept, Mecris context:** yebyen  
**Related session log:** 2026-09-06 Voice Input & Reasoning Test
OKF Knowledge Base Setup Complete - 2026-09-06: Initialized OKF v0.2 bundle, created 17 architectural concepts, validated all operations. OKF ready for agent workflow integration.
