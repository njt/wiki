# aispace

aispace is a Go CLI for "bot-friendly file drops" — temporary file storage built to be driven by AI agents rather than humans. It uploads files (or stdin) with a raw streaming POST, optionally mints separately-expiring public share links, and can encrypt payloads client-side with age X25519 so the storage operator never sees plaintext. It matters as a reference implementation of agent-native CLI design: stable exit codes, exact JSON, and a URL printed last on its own line make it trivially scriptable, and it fills the auth gap most agent-CLI frameworks leave open by treating bot keys and decryption identities as credentials.

---

## Architecture

A single binary, layered thin: `cmd/` (cobra subcommands) → `internal/api/` (the HTTP client, ~600 lines) → `internal/config` and `internal/duration`. The server is not in this repo — only the client, the agent skill, and examples. ~8.6K lines of Go including tests, with just two direct dependencies: `filippo.io/age` and `spf13/cobra`.

Each command returns a process exit code via `classify()` in `cmd/root.go:314`, which walks the error chain — `usageError` (exit 2), `codedError` (local errors like `unauthenticated`), then `api.Error` — and maps it onto codes 0–5. `internal/api/errors.go` maps stable server error codes (`invalid_key`, `quota_exceeded`, `monthly_upload_cap`) to exit codes, falling back to HTTP status for servers that don't return structured errors.

Uploads are raw bodies, not multipart: `POST /v1/files` with `X-File-Name`, `X-SHA256`, `X-File-Visibility`, and `X-Aispace-Encryption` headers (`internal/api/client.go:107`). The client's `Result[T]` pairs a decoded value with the raw JSON body so the CLI can print *exactly* what the API returned, with no re-serialization drift.

## Key techniques

- **Per-phase timeouts, no whole-request deadline.** `client.go:27` sets dial 30s / TLS 10s / header 60s / transfer-inactivity 2m. The comment explains why: a single cap over the body meant transfer time was really a bandwidth floor — a 100 MB upload had to sustain ~1.4 Mbit/s or die after transferring most of itself. Give up fast *before* bytes flow, stay patient *once they do*.
- **Reset-on-progress stall detection.** `internal/api/transfer.go` wraps the body in a `progressReader`/`watchedBody` that calls `watch.touch()` on every successful Read; a `time.AfterFunc` timer cancels the context with `ErrTransferStalled` when no bytes move for the inactivity window. A stall is classified as `timeout`, distinct from a network error.
- **Retry discrimination, not retry aggression.** `doRequest` retries idempotent GETs once — but only when the 429's error code is `rate_limited` (a monthly cap also arrives as 429, with `Retry-After` pointing at next month, so waiting is pointless), plus 502/503/504. 500 is deliberately excluded ("retrying only doubles the load"). Downloads are never replayed: a gateway error may arrive *after* the server recorded the download, so replaying could double-charge monthly allowance. `Retry-After` parsing handles both delta-seconds and HTTP-date, capped at 30s.
- **Client-side age X25519 encryption.** `cmd/crypto.go` encrypts into a temp file then streams it; the identity is written mode 0600 and never sent. Failure-mode-aware cleanup is the subtle bit: on a 4xx the server rejected the request so nothing was stored, and the identity is discarded; on a transport failure the file *may* have been stored, so the identity is kept with a warning. If no `--identity-out` is given, a recovery identity is auto-written to `~/.config/aispace/recovery/`.
- **Overflow-safe duration parsing.** `internal/duration/duration.go` accepts Go durations plus a `d` day suffix and bare seconds, with `scale()`/`add()` guards — a `time.Duration` spans ~292 years, and `1000000000000d` used to wrap to a valid-looking value.
- **Streaming SHA-256 verification.** `download --verify` hashes as it writes via `io.TeeReader` and compares against the digest recorded at upload; on mismatch the deferred cleanup removes the corrupt file before it can be trusted.

## Design decisions

- **The machine contract is the product; human output is a courtesy.** URL last on its own line, errors as JSON on stderr, stable exit codes. Everything is tuned for an agent's `subprocess.run` loop, not a human's terminal.
- **Correctness-of-failure beats retry aggressiveness.** Refusing to retry downloads and 500s trades raw resilience for never double-billing a metered account. A bot that replays a non-idempotent op is worse than one that fails.
- **Zero-knowledge posture.** Encryption happens locally; the server holds ciphertext only, and the decryption identity is treated as a credential (`AISPACE_AGE_IDENTITY` env var, deliberately *not* accepted as a flag so it can't leak into shell history).
- **Simplicity over flexibility.** Two dependencies, no framework, raw-body uploads that stream from disk with a known Content-Length. The trade-off is that everything about the *server* (retention, revocation, quotas) is a contract the client trusts rather than implements.

## Comparison notes

- Against [[10 Principles for Agent-Native CLIs]]: aispace is Chow's table-stakes tier realized concretely, and it closes the exact gap his framework leaves — auth in the agent context.
- Against [[Layer-First Pattern — Keep Data Out of the LLM Context]]: the file drop *is* the layer-first pattern at the transport layer; the acknowledgment payload is a URL plus expiry, not the data.
- Against [[Agent-Native Architectures (Every)]]: "files as universal interface" extended off-machine, with expiry, revocation, and per-link download caps added on top.
- Against [[AI-Ready APIs — Postman AWS Competency]]: the HTTP contract ships as both prose (`docs/API.md`) and executable Go types (`internal/api/types.go`), with stable machine-readable error codes.

#tool #project #agents #security #cli

---
*Sources: [[raw/aispace-client]], [[summary/aispace-client]]*
*Last updated: 2026-09-08*
