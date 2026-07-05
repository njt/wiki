---
url: https://github.com/emmayusufu/tessera
title: Tessera — Consent-gated remote access broker
author: Emma Yusufu
date_fetched: 2026-06-15
date_published: 2025-06-01
---

# Tessera — Deep Architecture Analysis

## Overview

Tessera is a consent-gated remote access broker written in Go (~5,000 lines). Three small binaries (coordinator, agent, tessera CLI) provide just-in-time, human-approved TCP forwarding and interactive shell access. The model: one person shares a local port by minting an 8-character code, another person redeems that code, the host types `y` at their terminal to approve, and a tunnel opens for the session. Nothing persists between sessions — no tokens, no accounts, no open ports.

The name derives from the Roman *tessera hospitalis*, a token given to a guest as proof of a trusted relationship.

## Repository structure

```
cmd/coordinator/main.go       — Broker entry point (103 lines)
cmd/agent/main.go              — Agent entry point (113 lines)
cmd/tessera/main.go            — Guest CLI entry point + CA/quickstart/connect (256 lines)
cmd/tessera/share.go           — Share command: mint code, upload bundle, approve loop (524 lines)
cmd/tessera/join.go            — Join command: redeem code, peek, service selection, forward (517 lines)
cmd/tessera/link.go            — Save/load coordinator config (144 lines)
cmd/tessera/kinds.go           — Service kind taxonomy + port inference (104 lines)
cmd/tessera/interactive.go     — Interactive mode (82 lines)
cmd/tessera/tail.go            — Audit log tailing
cmd/tessera/token.go           — Operator token management
cmd/tessera/colors.go          — Terminal color helpers
cmd/tessera/help.go            — Help text rendering
internal/coordinator/coordinator.go — Core broker: session management, request resolution (551 lines)
internal/coordinator/http.go        — HTTP endpoints: redeem, peek, revoke, healthz (221 lines)
internal/coordinator/bootstrap.go   — One-shot code minting with rejection sampling (246 lines)
internal/coordinator/share.go       — Share upload handler + validation (124 lines)
internal/coordinator/streams.go     — Guest/agent data stream pairing (80 lines)
internal/coordinator/ratelimit.go   — Per-IP + per-ID rate limiting (73 lines)
internal/agent/agent.go             — Host-side stream serving + shell mode (254 lines)
internal/client/client.go           — Guest-side request + forward (99 lines)
internal/proto/proto.go            — Length-prefixed JSON control protocol (83 lines)
internal/certs/certs.go            — ECDSA P-256 CA + cert issuance + TLS configs (149 lines)
internal/audit/audit.go            — Append-only JSONL audit log (75 lines)
internal/netutil/netutil.go        — Bidirectional pipe with idle timeout (47 lines)
```

## Architecture

### Three-Component Topology

```
Guest (tessera CLI) ──mTLS──▶ Coordinator (broker) ◀──mTLS (dials out)── Agent
                                  │                                        │
                            HTTP :8080                              TCP → local resource
                         /redeem /peek /revoke
```

**Coordinator** (`internal/coordinator/coordinator.go`): Runs on a public host. Accepts mTLS connections from agents and guests, relays approved streams as opaque ciphertext (inner TLS terminates at endpoints), and writes an append-only audit log. Owns all session state: `agents`, `requests`, `sessions`, `pending`, `approvers`, `sharePins`, `sessionTimeouts` — all protected by a single `sync.Mutex`.

**Agent** (`internal/agent/agent.go`): Runs on the host's side as a background service. Dials out to the coordinator and waits — accepts nothing inbound. On an approved stream it connects to the local `host:port` (or spawns a PTY for -shell mode) and pipes through inner TLS. Has exponential backoff reconnect (1s→30s with jitter). Supports session recording for shell mode.

**Tessera CLI** (`cmd/tessera/`): The guest CLI. Subcommands: `ca` (mint CA), `quickstart` (local dev setup), `connect` (direct share-id connection), `join` (redeem a share code), `share` (offer a port/service/shell), `token` (save operator token), `link` (save coordinator config).

### Protocol

Control messages are length-prefixed JSON frames (4-byte big-endian uint32 length header, max 64KB). Eleven message kinds defined in `internal/proto/proto.go`:

- `register` — Agent announces itself to coordinator
- `request` — Guest requests access to a share
- `decision` — Coordinator returns approval/denial to guest
- `open_data` — Coordinator tells agent to open a data stream
- `data_hello` — Guest or agent announces a data connection (carries ConnID or SessionID)
- `approval_subscribe` — Host terminal subscribes for approval prompts
- `approval_prompt` — Coordinator sends approval request to host
- `approval_decision` — Host responds y/n
- `share_upload` — Host uploads a bootstrap bundle
- `share_response` — Coordinator returns the minted code
- `session_ended` — Coordinator notifies host that a session closed

### Session lifecycle

1. Agent dials out → coordinator, sends `register` with share_id (blocking read loop keeps connection alive)
2. Host runs `tessera share` → mints one-shot guest certs, uploads bootstrap bundle via `share_upload`, receives 8-char code
3. Guest runs `tessera join CODE` → HTTP `GET /peek/{code}` for metadata, then `POST /redeem/{code}` to get guest cert + CA + coordinator address (one-shot — code consumed)
4. Guest dials coordinator with `request` message
5. Coordinator fans out `approval_prompt` to all subscribed approver connections for that share_id
6. Host sees prompt at terminal: name, services, reason → types `y` or `n`
7. On approval: guest receives `decision` with `approved:true` + session_id; coordinator creates session
8. Guest opens local listener, accepts connection, dials coordinator data channel (`data_hello` role=guest)
9. Coordinator sends `open_data` to agent with conn_id, pairs via a `pending` map (chan-based rendezvous)
10. Guest and agent perform inner TLS handshake end-to-end through the coordinator
11. Bidirectional pipe with idle timeout (default 30 min) via `netutil.PipeIdle`
12. Either side disconnects → coordinator closes all session streams, notifies approvers

### End-to-end encryption

The critical design: inner TLS terminates at guest and agent, NOT at the coordinator. The coordinator relays only ciphertext. The integration test `TestEndToEndEncryptedForward` (`internal/coordinator/integration_test.go:180`) verifies this by recording all bytes on the relay path and asserting a plaintext marker never appears.

## Key Techniques

### Certificate pinning by share-id

The coordinator's `sharePins` map (`coordinator.go:117`) binds each share_id to the SHA-256 fingerprint of the first certificate that registers or uploads for it. Subsequent attempts with a different cert are rejected with a `share-id owned by a different cert` error. This prevents an attacker who compromises the coordinator's HTTP endpoint from impersonating the host — they'd need the host's private key.

### One-shot bootstrap codes with rejection sampling

`internal/coordinator/bootstrap.go:213-237` generates 8-character codes from a 30-character alphabet (`23456789ABCDEFGHJKMNPQRSTVWXYZ` — omitting visually ambiguous characters like 0/O, 1/I/L). Uses rejection sampling to ensure uniform distribution: draws random bytes, accepts only those below 240 (the largest multiple of 30 below 256), so every character has exactly equal probability. Plain `byte % 30` would bias 16 characters to 9/256 and 14 to 8/256.

Codes are stored as SHA-256 hashes in the audit log — the raw code is never persisted.

### Peek-before-redeem

The guest CLI first calls `GET /peek/{code}` to see service names, host name, and reason WITHOUT consuming the code. Only `POST /redeem/{code}` triggers one-shot consumption. This lets the guest see what they're connecting to before committing. The peek response intentionally omits certs, targets, and the coordinator address.

### Dual rate limiter

`internal/coordinator/ratelimit.go`: Two-layer rate limiting. Per-IP: 5 requests/minute using a sliding window. Per-ID (code or session): 10 failed attempts triggers a 5-minute lockout. Both write `rate_limit_trip` audit events. The healthz endpoint deliberately bypasses rate limiting so uptime monitors don't trip it.

### Service kind inference

`cmd/tessera/kinds.go` maps well-known ports (5432→postgres, 6379→redis, 22→SSH, etc.) to service kinds, then renders guest-side connection hints (e.g., `psql -h 127.0.0.1 -p {port}`). The hint is generated locally on the guest, not taken from a host-supplied string — preventing injection.

### Agent reconnection with jitter

`agent.RunWithBackoff` (`agent/agent.go:45-67`): Exponential backoff 1s→2s→4s→8s→30s with random jitter (`±delay/8`) to avoid thundering herd on coordinator restart. Uses `math/rand` (not crypto/rand) since this is for timing, not security.

### Audit field capping

`internal/audit/audit.go:42-49`: Every user-controllable field (who, target, reason, detail) is capped at 256 bytes before writing to the audit log. Prevents a malicious guest from bloating the log with a 10MB reason string. Each write calls `f.Sync()` for durability.

### ANSI escape sanitization

Both host and guest sanitize counterparty-supplied strings (`cmd/tessera/share.go:377-392`): strips ASCII control bytes (0x00–0x1F, 0x7F), replaces with `?`, caps at 200 chars. Prevents a malicious peer from smuggling ANSI escapes into the other's terminal.

### PTY size header clamping

`internal/agent/agent.go:238-253`: The guest sends an 8-byte header (rows, cols as uint32 big-endian) before shell data. The agent clamps values to 9999 before downcasting to uint16, preventing wrap-around attacks where `rows=70000 → uint16(4464)`.

### Terminal-as-approval-interface

Approval happens at the host's terminal over the SAME mTLS channel that carries all other traffic. There is no web endpoint for approval — no phishing vector, no MITM surface. The host subscribes via `KindApprovalSubscribe` on the mTLS connection, receives prompts, and sends back y/n decisions.

## Design Decisions

### Optimized for: Zero persistent state

Nothing lives between sessions. No user accounts, no API keys, no stored tokens. The bootstrap code expires (10s–600s TTL). The session dies when either side disconnects. The coordinator keeps everything in memory — restart it and all state is gone except the audit log. This is the core design philosophy: the tool should leave no trace except the audit trail.

### Optimized for: No inbound connections to the host

The agent dials OUT. The host's firewall never needs an inbound rule. This is the key operational property that makes Tessera usable from behind NAT, in coffee shops, on corporate VPNs — anywhere the agent can reach the coordinator. It's the same design pattern as ngrok's agent, but with consent gating and mTLS instead of ngrok's SaaS auth model.

### Optimized for: Cryptographic identity over passwords

Everything is certificate-based. The coordinator requires mutual TLS. Guest sessions are bound to the guest's certificate fingerprint — a different enrolled guest cannot ride someone else's session even with the session ID. The `TestSessionHijackRejected` test (`integration_test.go:233`) verifies this: an attacker with a different cert who knows the session ID gets their connection silently dropped.

### Sacrificed: Scale and multi-tenancy

The coordinator is a single process with in-memory state. No sharding, no horizontal scaling, no persistence layer beyond the audit log. This is fine for small teams (the stated use case) but would need a redesign for hundreds of concurrent sessions. The `sync.Mutex` on the Coordinator struct is the single contention point.

### Sacrificed: UX polish for audit integrity

The approval interface is a raw terminal prompt. No web UI, no mobile push notification, no Slack integration. The design explicitly keeps the approval path on the same mTLS channel to avoid introducing phishing vectors. This is a security-first choice that trades convenience for integrity.

### Sacrificed: Feature breadth for simplicity

Compared to Teleport (which the README explicitly acknowledges), Tessera doesn't do SSH certificate authorities, Kubernetes access, session recording with replay, RBAC, or SSO integration. It does exactly one thing: TCP forward with human approval and an audit log. The README's honesty about this ("if you can run Teleport, use it") is characteristic.

## Comparison Notes

**vs Teleport**: Teleport's free Community Edition does far more (SSH, K8s, DBs, RDP, SSO, session recording). Tessera's sole advantage is the human-approve, just-in-time access flow — which Teleport gates behind Enterprise. Tessera is MIT-licensed vs Teleport's AGPL. For a 3-person team that just needs "let Alice hit my postgres for 15 minutes," Tessera is simpler to deploy (one binary, no cluster).

**vs ngrok**: ngrok is SaaS with persistent tunnels, auth tokens, and a web dashboard. Tessera is self-hosted, session-scoped, and consent-gated. ngrok is for exposing services persistently; Tessera is for granting temporary access.

**vs Tailscale**: Tailscale is a full mesh VPN with persistent identity. Tessera is a point-to-point, session-scoped tunnel with no persistent network membership. Tailscale's "Tailscale Serve" / "Funnel" are closer comparables but still require shared tailnet membership.

**vs SSH tunneling**: `ssh -R` gives you a reverse tunnel but requires the guest to have an SSH account on the host (or the host on the guest's machine). Tessera provides anonymous guest access with cryptographic identity binding and an audit trail — you don't need to create an account for the person you're helping.

**vs Akmon** (in wiki): Akmon provides tamper-evident audit for AI agent sessions. Tessera provides consent-gated access with audit for human-to-human resource sharing. Both care about cryptographic proof of what happened, but at different layers: Akmon records agent actions, Tessera records access grants.

## Security Properties (verified by tests)

- End-to-end encryption: plaintext never appears on the relay path (`TestEndToEndEncryptedForward`)
- Session hijack prevention: different cert rejected for same session ID (`TestSessionHijackRejected`)
- Revoke closes live streams immediately (`TestRevokeClosesLiveStream`)
- Request denial works correctly (`TestRequestDenied`)
- No agent = clean error, not a hang (`TestRequestNoAgent`)

## Dependencies

Minimal by design. Only two external dependencies beyond stdlib:
- `github.com/creack/pty` — PTY support for shell mode
- `golang.org/x/term` — Terminal raw mode + size detection for shell sessions

No framework, no ORM, no web framework (uses `net/http` stdlib). The entire dependency tree fits in a 18MB distroless Docker image.

## Code quality observations

- Single `sync.Mutex` on Coordinator — simple but becomes the bottleneck under load
- Error handling is explicit and uses sentinel values, not custom error types
- Tests use real TLS connections, not mocks — the integration test spins up a real coordinator, agent, and echo server
- Pre-commit hooks enforce: no em-dashes, no AI/Claude attribution in commits, no work-email leakage (author is protective of their personal project identity)
- Request timeout is 5 minutes (line 331 of coordinator.go) — generous for human-in-the-loop approval
- Stream pairing uses a 15-second rendezvous timeout (`pairTimeout`) — if the agent doesn't claim the stream within 15s, the guest side is closed
