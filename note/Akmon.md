# Akmon

Akmon is a **tamper-evident evidence and verification layer for AI agents**, built as a ~95K-line Rust workspace. It turns any AI coding agent session — either its own built-in agent, or any OpenTelemetry-instrumented agent — into a portable, content-addressed, cryptographically signed record (the AGEF format). A third party can verify that record offline with nothing but `openssl`. It's also a fully functional coding agent with a terminal UI, but the evidence pipeline is the product.

The name comes from the ancient Greek word for "anvil" — shaping difficult work with pressure and precision, while maintaining control over every strike.

## Architecture

Akmon is a **14-crate Rust workspace** organized into four layers:

**Foundation** — `akmon-core` provides the FSM (8 discrete states: Idle → Planning → Thinking → ToolExecution → AwaitingConfirmation → Summarizing → Complete/Failed), a **deny-by-default policy engine** with five modes (DenyAll, Interactive, AutoApproveReads, AutoApproveReadsAndFetch, Configured), a symlink-aware filesystem sandbox with multi-root support, audit event logging, and deterministic replay metadata hashing.

**Storage** — `akmon-journal` implements the AGEF substrate: a content-addressed, merkle-linked event chain backed by `redb` (embedded Rust DB). Events are serialized with `postcard` for compact storage and canonical CBOR (`ciborium`) for deterministic hashing. Each event has parent hashes, creating an append-only tamper-evident chain. `akmon-bundle` packages sessions into `tar.zst` archives with `manifest.json`, `events.bin`, and a content-addressed `objects/` directory.

**Agent** — `akmon-models` provides an `LlmProvider` trait with backends for Anthropic (`anthropic.rs`, 1703 lines), OpenAI-compatible (`openai_compat.rs`, 1769 lines), Ollama (`ollama.rs`, 1746 lines), and AWS Bedrock (`bedrock.rs`, 1222 lines). `akmon-tools` defines the `Tool` trait with 20+ built-in tools. `akmon-query` owns the main session loop (`session.rs`, 7030 lines) with context management, auto-compaction at 85% context utilization, micro-compaction trimming, and parallel tool dispatch with policy gating.

**Verification** — `akmon-replay` does deterministic session re-execution using `PlaybackProvider`/`PlaybackTool` to substitute stored responses instead of live LLM calls. `akmon-diff` compares two sessions event-by-event. `akmon-otel` imports OpenTelemetry GenAI traces into AGEF sessions. `agef-verify` is a standalone binary (~700 lines) that verifies bundles without needing the full Akmon install.

## Key Techniques

**Canonical CBOR for deterministic content-addressing.** Events are serialized to canonical CBOR with sorted keys, CBOR tag 1 timestamps, and unified hash representation — so identical logical content always hashes identically, regardless of platform or serialization order. This is what makes the merkle chain sound.

**Decorator pattern for transparent journaling.** `JournalingProvider` wraps every LLM provider at session construction, storing request/response objects and emitting `ProviderCall` events. `JournalingTool` wraps every tool, storing input/output and emitting `ToolCall` events. Neither the provider nor the tool needs to know about journaling.

**Capture level baked into the signed head.** The capture level (`full` for byte-for-byte replayable sessions, `structural` for OTEL imports with only metadata) is stored in the session config, hashed into the SessionStart event, which feeds the merkle chain head that gets signed. Nobody can upgrade `structural` to `full` without breaking the signature.

**Ed25519 detached signatures with PKCS#8 v2.** Signing keys are standard PKCS#8 v2. The `prove-openssl` command extracts SPKI public key PEM, statement bytes, and raw signature — verifiable with `openssl pkeyutl -verify` on any machine with OpenSSL 3.x. No Akmon install needed.

**Dual budget/iteration limits.** The agent loop checks both `max_iterations` (hard ceiling) and `max_budget_usd` (cumulative cost from provider usage reports). The budget cap prevents starting another model call but lets the current batch of tools complete cleanly.

**Multi-attempt provider calls with journaled retries.** Each `ProviderCall` event records every attempt (request hash, response hash if present, attempt status from Success through RateLimited/Cancelled/ClientError, timing, error messages). This means the evidence chain captures transient failures transparently.

## Design Decisions

**Optimized for verifiability, not throughput.** Canonical CBOR serialization and content-addressing add overhead per event. This is the right trade-off for evidence that must survive skeptical third-party scrutiny years later.

**Deny-by-default.** The default `DenyAll` policy means the agent can do nothing without explicit configuration. More friction than permissive-default tools like Claude Code, but correct for an evidence system.

**Embedded database over SQLite.** Uses `redb` + `postcard` rather than SQLite. Zero-config deployment, no schema migrations, compact binary encoding. Trade-off: no SQL queries, but the evidence model doesn't need them.

**One session = one journal.** No cross-session query or global index. Each session is independently verifiable. Cross-session analysis is handled by `akmon diff` and `akmon replay` operating on specific pairs.

**Semantic index is optional.** Behind a `semantic-index` feature flag because `fastembed` pulls in ONNX runtime. The core evidence pipeline works without it.

**TUI is a consumer, not a dependency.** The terminal UI communicates with the session via channels (`mpsc`). The same event stream could feed a headless CLI, a web UI, or an automated harness.

## Comparison Notes

Unlike [[claude-replay]] which visualizes what happened, Akmon **re-runs the session deterministically** with playback providers/tools and compares every event — a much stronger verification guarantee.

Unlike [[Zeroclaw]] (another Rust agent with 31k stars, focused on channel diversity and OS-level sandboxing), Akmon's differentiator is the evidence layer. The agent is a reference producer for the AGEF format, not the destination.

Unlike Microsoft's Agent Governance Toolkit (hash chain + HMAC, no asymmetric signatures, Azure-tied), Akmon uses **Ed25519 asymmetric signatures** with standalone offline verification via `openssl`. It complements Microsoft's stack — can seal Purview captures and verify Foundry traces.

Unlike [[State System]] (Python, model-mediated state interpretation), Akmon uses **deterministic byte-level content-addressing** with canonical CBOR. Where State System asks "what does the model think happened?", Akmon asks "what are the exact bytes, and can you verify them?"

The AGEF format is separately specified at `github.com/radotsvetkov/agef` (v0.1.3). Akmon is the reference implementation, but the format is designed to be implementable by others.

## Tags

#tool #agent #security #audit #rust #evidence #tamper-evident #verification

## Related Pages

- [[Security and Sandboxing]] — Hub for agent isolation and credential management
- [[State System]] — Organizational state layer with evidence-first commits
- [[Zero Trust for AI Agents]] — Anthropic's security framework for agents
- [[Guardrails and Feedback Loops]] — Deterministic enforcement over prompts
- [[Zeroclaw]] — Another Rust agent runtime, focused on channel/sandbox architecture
- [[Components of a Coding Agent]] — Harness vs model taxonomy
- [[Agent Identity]] — Memory as retrieval vs identity as participation
- [[Agent Memory and Context]] — Context management as an engineering challenge
- [[claude-replay]] — Agent sessions as embeddable HTML replays
- [[session-analysis]] — Analyze agent session JSONL for tokens and cost
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management over model ability
- [[Golem Covenant]] — Bounded, answerable, revocable agents specification

---

*Source: [github.com/radotsvetkov/akmon](https://github.com/radotsvetkov/akmon) (radotsvetkov). Apache-2.0. Fetched 2026-06-09 via full repository clone and source analysis.*
