# Tessera — Consent-gated remote access broker

Tessera is a Go tool (~5K lines) for just-in-time, human-approved TCP forwarding. One person shares a local port and gets an 8-character code; another redeems it; the host types `y` at their terminal; a tunnel opens for the session. Nothing persists — no accounts, no tokens, no open ports. Three small binaries: a coordinator (broker), an agent (host-side daemon that dials out so nothing accepts inbound connections), and a CLI. MIT-licensed, by Emma Yusufu.

## Architecture

Three Go binaries connected by mutual TLS:

- **Coordinator** (`internal/coordinator/coordinator.go`, 551 lines): Runs on a public host. Accepts mTLS agent/guest connections and serves HTTP for code redeem/peek. Owns all in-memory session state (`agents`, `requests`, `sessions`, `pending`, `approvers`) behind a single `sync.Mutex`. Relays approved streams as opaque ciphertext — inner TLS terminates at the endpoints, not here.

- **Agent** (`internal/agent/agent.go`, 254 lines): Runs on the host side. Dials OUT to the coordinator — never accepts inbound connections (works behind NAT). On approved streams, connects to the allowed local `host:port` or spawns a PTY for shell mode (`-shell`). Exponential backoff reconnect (1s→30s with jitter) survives network flaps. Supports session recording for shell sessions.

- **Tessera CLI** (`cmd/tessera/`): Guest and host interface. Subcommands: `ca` (mint PKI), `quickstart` (local dev setup), `connect` (direct share-id), `join` (redeem a code), `share` (offer port/service/shell), `token`/`link` (config).

**Protocol** (`internal/proto/proto.go`, 83 lines): Length-prefixed JSON frames (4-byte uint32 BE header, max 64KB). Eleven message kinds: register, request, decision, open_data, data_hello, approval_subscribe/prompt/decision, share_upload/response, session_ended.

**Session flow**: Agent registers → Host uploads bootstrap bundle → Guest redeems one-shot code (HTTP) → Guest requests access (mTLS) → Host approves at terminal (same mTLS channel) → Guest + Agent perform inner TLS handshake end-to-end → bidirectional pipe with 30-min idle timeout. Either disconnect ends everything.

## Key Techniques

**Certificate pinning by share-id**: The coordinator's `sharePins` map binds each share_id to the SHA-256 of the first cert that claims it. Later attempts with a different cert are rejected — prevents impersonation even if the HTTP endpoint is compromised.

**One-shot bootstrap codes with rejection sampling** (`bootstrap.go:213-237`): 8-char codes from a 30-char alphabet (visually unambiguous: no 0/O, 1/I/L). Draws random bytes, accepts only those below 240 (largest multiple of 30 under 256) so every character has exactly equal probability. `byte % 30` would bias the distribution.

**Peek-before-redeem**: `GET /peek/{code}` returns service names/host/reason WITHOUT consuming the code. `POST /redeem/{code}` is one-shot. Peek response intentionally omits certs and coordinator address.

**End-to-end encryption verification**: `TestEndToEndEncryptedForward` records all bytes on the relay path and asserts the plaintext marker never appears — the coordinator truly sees only ciphertext.

**Dual rate limiter** (`ratelimit.go`): 5 req/min per IP (sliding window) + 10 failed attempts per ID → 5-min lockout. Healthz endpoint bypasses it. Both fire audit events.

**Terminal-as-approval-interface**: Approval happens over the SAME mTLS channel as everything else. No web endpoint, no phishing vector, no out-of-band link. The host types `y` or `n` at their terminal.

**Service kind inference** (`kinds.go`): Maps well-known ports to service kinds (5432→postgres, etc.), generates guest-side connection hints locally — from a closed enum, never from a host-supplied string (prevents injection).

**Audit field capping**: All user-controllable fields capped at 256 bytes before writing to the append-only JSONL audit log. Each write calls `f.Sync()`.

**ANSI escape sanitization**: Both sides strip control bytes from counterparty-supplied strings — prevents terminal injection attacks.

## Design Decisions

**Optimized for zero persistent state**: No accounts, no API keys, no stored tokens. Bootstrap codes expire (10s–600s TTL). Sessions die on disconnect. Everything is in memory except the audit log.

**Optimized for no inbound connections**: The agent dials OUT. Works from behind NAT, coffee shops, corporate VPNs. Same pattern as ngrok's agent but with consent gating and self-hosted mTLS instead of SaaS auth.

**Optimized for cryptographic identity**: Everything is certificate-based. Guest sessions are bound to the guest's cert fingerprint — a different enrolled guest can't ride someone else's session even with the session ID (verified by `TestSessionHijackRejected`).

**Sacrificed scale for simplicity**: Single process, in-memory state, one `sync.Mutex`. No sharding, no persistence layer beyond audit. Fine for small teams; would need redesign for hundreds of concurrent sessions.

**Sacrificed UX polish for audit integrity**: Raw terminal prompt for approval. No web UI, no push notification, no Slack. The approval path stays on the mTLS channel — no phishing surface.

## Comparison Notes

**vs Teleport**: Teleport's free tier does far more (SSH, K8s, DBs, RDP, SSO, session recording). Tessera's sole advantage is the human-approve, just-in-time flow that Teleport gates behind Enterprise. MIT vs AGPL. For a small team that just needs "let Alice hit my postgres for 15 minutes," Tessera is dramatically simpler to deploy — one binary, no cluster.

**vs ngrok**: ngrok is SaaS with persistent tunnels and auth tokens. Tessera is self-hosted, session-scoped, and consent-gated. Different use case: ngrok exposes services persistently; Tessera grants temporary access.

**vs Tailscale**: Full mesh VPN with persistent identity vs point-to-point session-scoped tunnel. Tailscale Serve/Funnel are closer but still require shared tailnet membership.

**vs SSH tunneling**: `ssh -R` requires the guest to have an SSH account. Tessera provides anonymous guest access with cryptographic identity binding and an audit trail.

**vs [[Akmon]]**: Both care about cryptographic proof of what happened. Akmon records agent actions; Tessera records access grants.

## Tags

#tool #security #networking #go #tunneling #access-control #mtls #zero-trust

## Source

GitHub: [emmayusufu/tessera](https://github.com/emmayusufu/tessera) — fetched 2026-06-15 via full repo clone and source-level analysis.
