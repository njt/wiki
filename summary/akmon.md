---
url: https://github.com/radotsvetkov/akmon
title: Akmon
author: radotsvetkov
date_fetched: 2026-06-09
date_published: 2025
topics:
  - misc
---

# Akmon — Full Repository Analysis

## Project Overview

Akmon is a **tamper-evident evidence and verification layer for AI agents**, implemented as a ~95K LoC Rust workspace across 14 crates. It is also a fully functional AI coding agent with a terminal UI, but the core value proposition is the evidence pipeline: any agent session (Akmon's own, or any OpenTelemetry-instrumented agent) becomes a portable, content-addressed, cryptographically signed record that can be verified offline with standard `openssl` — no Akmon install, no cloud service, no trust required.

The name comes from the ancient Greek word for "anvil" — the idea is shaping difficult work with pressure and precision while maintaining control over every strike (permissions, audit trail, evidence).

## Repository Structure

```
akmon/
├── Cargo.toml              # Workspace (14 crates)
├── AKMON.md                # Mini architecture overview
├── POLICY_SYSTEM_EXPLANATION.md  # Detailed policy docs
├── README.md               # Full project description
├── docs/                   # mdBook documentation site
├── landing/                # Landing page HTML
├── packaging/homebrew/     # Homebrew formulae
├── scripts/                # CI/doc validation scripts
├── crates/
│   ├── akmon-core/         # Foundation: FSM, policy, sandbox, secrets, audit, replay
│   ├── akmon-config/       # YAML/TOML config loading (global + per-project)
│   ├── akmon-models/       # LLM provider abstraction (Anthropic, OpenAI, Ollama, Bedrock)
│   ├── akmon-tools/        # 20+ built-in tools with journaling
│   ├── akmon-query/        # Agent session loop, context management
│   ├── akmon-index/        # Semantic embedding index (BGESmallENV15 via fastembed)
│   ├── akmon-journal/      # Content-addressed merkle event store (AGEF substrate)
│   ├── akmon-bundle/       # AGEF bundle format: tar.zst, manifest, signing, verification
│   ├── akmon-replay/       # Deterministic session replay engine
│   ├── akmon-diff/         # Session comparison/diff engine (two sessions side by side)
│   ├── akmon-otel/         # OpenTelemetry GenAI trace → AGEF session import
│   ├── akmon-cli/          # Binary entry point (7255 lines), all CLI commands
│   ├── akmon-tui/          # Ratatui-based terminal UI (crossterm + ratatui)
│   └── agef-verify/        # Standalone verifier binary (706 lines), no full Akmon needed
```

## Key Source Files (line counts)

| File | Lines | Purpose |
|------|-------|---------|
| crates/akmon-cli/src/main.rs | 7255 | CLI entry point, all subcommands |
| crates/akmon-query/src/session.rs | 7030 | Core agent session loop |
| crates/akmon-replay/src/engine.rs | 2793 | Deterministic replay orchestration |
| crates/akmon-tui/src/runner.rs | 1998 | Terminal UI runner/session integration |
| crates/akmon-diff/src/engine.rs | 1980 | Session comparison engine |
| crates/akmon-cli/src/bundle_cmd.rs | 1969 | Bundle create/sign/verify/export commands |
| crates/akmon-models/src/openai_compat.rs | 1769 | OpenAI-compatible provider |
| crates/akmon-models/src/ollama.rs | 1746 | Ollama backend |
| crates/akmon-models/src/anthropic.rs | 1703 | Anthropic backend |
| crates/akmon-journal/src/session_graph.rs | 1576 | Merkle graph operations |
| crates/akmon-models/src/journaling.rs | 1548 | JournalingProvider decorator |
| crates/akmon-otel/src/lib.rs | 1488 | OTEL GenAI → AGEF mapper |
| crates/akmon-core/src/policy.rs | 1218 | Policy engine |

## Architecture Deep Dive

### 1. The Evidence Format (AGEF)

AGEF (AI Agent Evidence Format) is the foundation. Every session is a **content-addressed, merkle-linked event chain**:

- **Events** are the core unit. Each has `parents: Vec<Hash>`, `kind: EventKind`, `emitted_at: OffsetDateTime`, `sequence: u64`
- **EventKind variants**: SessionStart, UserTurn, ProviderCall, ToolCall, RetrievalCall, PermissionGate, AssistantTurn, SessionEnd
- **Hashing**: Canonical CBOR via `ciborium` ensures deterministic serialization. Hashes use either SHA-256 or BLAKE3
- **Objects**: All content (prompts, responses, tool I/O, configs) stored in a content-addressed object store by hash
- **Journal**: Backed by `redb` (embedded Rust DB) with `postcard` for compact serialization
- **Bundle**: `tar.zst` archive containing `manifest.json`, `events.bin` (length-delimited canonical CBOR frames), and `objects/<hex>` directory

The critical insight: Wire events use canonical CBOR where all `Hash` fields become `WireHash` (algorithm+bytes array), timestamps become CBOR tag 1, and keys are sorted — so identical logical content **always** produces identical hashes regardless of serialization quirks.

### 2. The Session Loop (AgentSession)

Located in `crates/akmon-query/src/session.rs`, the `AgentSession` struct owns the entire agent lifecycle:

- **State machine**: Idle → Planning → Thinking → ToolExecution → AwaitingConfirmation → Summarizing → Complete/Failed
- **`run()` method** (line 865): Contains the main `'session` loop with:
  - Iteration limit checking (default 25, configurable)
  - Budget cap (headless USD limit)
  - **Micro-compaction**: Trims context before each iteration for non-Groq/non-Ollama providers to keep recent N messages
  - **Auto-summarization**: When estimated tokens exceed 85% of available context window (with 30K reserve buffer), triggers context summarization pass (max 8 rounds)
  - **Message trimming**: For Ollama, keeps only system + last 6 non-system messages. For others, uses `context_limit_for_model()`
  - Provider call → tool dispatch → policy evaluation → parallel tool execution

### 3. Policy Engine (Deny-by-Default)

Located in `crates/akmon-core/src/policy.rs` (1218 lines):

**Five modes**: DenyAll → Interactive → AutoApproveReads → AutoApproveReadsAndFetch → Configured

**Permission types**: ReadFile, ListDirectory, WriteFile, ExecuteCommand, NetworkFetch

**Evaluation flow**:
1. Tool request → `concrete_permissions()` extracts typed permissions from tool args
2. `evaluate_automatic()` checks against current mode
3. If InteractiveRequiresCaller → emit ConfirmationRequired event, wait for PolicyVerdict from caller
4. If denied → skip execution, record denial message
5. If allowed → add to approved batch

**Configured mode** supports declarative rules for filesystem paths, shell prefixes, network domains, and specific tool names. Explicit deny always overrides allow; most-specific match wins.

### 4. Tool System

Located in `crates/akmon-tools/`. The `Tool` trait (`lib.rs`) defines:

```rust
pub trait Tool: Send + Sync {
    fn name(&self) -> &str;
    fn description(&self) -> &str;
    fn required_permissions(&self) -> &[Permission];
    fn parameters_schema(&self) -> JsonValue;
    async fn execute(&self, args: JsonValue, ctx: &ToolContext) -> ToolOutput;
    fn mcp_policy_context(&self) -> Option<McpPolicyContext>;
    fn side_effects_manifest(&self, _input: &Value, _output: &ToolOutput) -> Option<Vec<u8>>;
}
```

**Key design**: The `side_effects_manifest()` method allows tools to report canonical CBOR bytes describing observable side effects — these get hashed and stored in the journal's `ToolCall` event, making side effects part of the tamper-evident chain.

**JournalingTool wrapper** (`journaling.rs`): Every tool is transparently wrapped at session construction. This decorator intercepts every `execute()` call, stores input/output as objects, and emits `ToolCall` events in the merkle chain — without any tool needing to know about journaling.

### 5. Provider Architecture

`crates/akmon-models/` implements the `LlmProvider` trait (async):

```rust
pub trait LlmProvider: Send + Sync {
    fn name(&self) -> &str;
    fn context_window_tokens(&self) -> usize;
    fn completion_model_id(&self) -> &str;
    fn estimate_tokens(&self, messages: &[Message]) -> Option<usize>;
    async fn complete(&self, messages: &[Message], config: &CompletionConfig) -> Result<CompletionStream, ModelError>;
}
```

**Backends**: Anthropic (`anthropic.rs` 1703 lines), Ollama (`ollama.rs` 1746 lines), OpenAI-compatible (`openai_compat.rs` 1769 lines), Bedrock (`bedrock.rs` 1222 lines)

**JournalingProvider wrapper**: Like JournalingTool, transparently wraps any provider, storing request/response objects and emitting `ProviderCall` events with attempt records (including retries, rate limits, errors).

### 6. Replay Engine

`crates/akmon-replay/src/engine.rs` (2793 lines): Deterministic playback of full-capture sessions.

**Key mechanism**:
- `PlaybackProvider`: Reads stored provider responses from the object store and feeds them back instead of calling the LLM
- `PlaybackTool`: Reads stored tool outputs instead of executing tools
- **Two comparison modes**: Default (index lockstep, comparing event kinds and semantic content) and Strict (normalized projection hashes)
- `ReplayDivergenceCollector` tracks mismatches: MissingReplayEvent, UnexpectedReplayEvent, EventCountMismatch, ContentMismatch

The replay engine constructs a complete `AgentSession` with the same config, policy, tools, and sandbox, but substitutes playback providers/tools. The session runs its full loop, and every event is compared against the source session.

### 7. Diff Engine

`crates/akmon-diff/src/engine.rs` (1980 lines): Compares two independent sessions (from the same or different journals) and reports divergences. Unlike replay (which re-executes deterministically), diff is purely structural — it loads both session histories and compares event-by-event.

### 8. OpenTelemetry Import

`crates/akmon-otel/src/lib.rs` (1488 lines): Converts OTLP/JSON GenAI traces into AGEF sessions.

- Supports both semconv v1.37+ **structured** form (gen_ai.input.messages with role+parts) and legacy v1.36 **message-event** form (individual gen_ai.*.message span events)
- Legacy events are reduced to the same canonical structured JSON for consistent hashing
- Synthetic SessionStart/SessionEnd events bracket imported spans
- **Capture level** is honestly recorded: `Full` when real content is present, `Structural` when only metadata
- `akmon bundle verify --require-capture full` fails on structural-only imports

### 9. Semantic Index

`crates/akmon-index/`: Uses `fastembed` (Rust bindings for BGESmallENV15) to build embedding indices of project files. Respects `.gitignore`, `.akmonignore`, and skips common dependency directories. Persisted to `.akmon/index.bin` via `bincode`. Results sorted by cosine similarity.

### 10. Sandbox

`crates/akmon-core/src/sandbox.rs`: Path resolution with symlink-aware boundary enforcement. Supports multiple allowed roots (for monorepo siblings). All paths go through `sandbox.resolve()` which canonicalizes and checks containment after symlink resolution — path traversal (`../../../etc/passwd`) and symlink escapes are both caught.

## Key Techniques

### Canonical CBOR for Deterministic Content-Addressing
The wire format for events uses `ciborium` with sorted keys, CBOR tag 1 timestamps, and unified `WireHash` representation. This ensures that two events describing the same logical content always hash to the same value — critical for merkle chain integrity and cross-platform determinism.

### Decorator Pattern for Transparent Journaling
Both `JournalingProvider` and `JournalingTool` use the decorator pattern. The session wraps every provider and every tool at construction time, so every API call and every tool execution is automatically journaled without modifying the underlying provider or tool code.

### Capture Level Baked into Signed Head
The capture level (`full` vs `structural`) is stored in the session config object, which is hashed into the SessionStart event, which feeds the merkle chain head, which is signed. This means the capture level is **cryptographically bound** to the session — nobody can modify it later without breaking the signature.

### Ed25519 with PKCS#8 v2 for Offline Verification
Signing uses PKCS#8 v2 (RFC 5958) Ed25519 keys. The `prove-openssl` command extracts the SPKI public key as PEM, the statement bytes, and the signature — all checkable with `openssl pkeyutl -verify`. This is the core trust proposition: verification requires nothing but OpenSSL 3.x.

### Budget and Iteration Dual Limits
The agent loop checks two ceilings: `max_iterations` (hard iteration count) and `max_budget_usd` (cumulative cost estimate from provider usage reports). The budget cap sets `budget_stop_before_next_iteration` which prevents starting another model call — but lets the current batch of tools complete cleanly.

### Manual/Semantic Index Search
The `SearchTool` does text search (grep) while `SemanticSearchTool` (behind `semantic-index` feature flag) uses the embedded index. The indexer uses the `ignore` crate which respects `.gitignore` natively, plus an additional extension allowlist and blocklist.

## Design Decisions & Trade-offs

### Optimized for verifiability, not speed
The canonical CBOR serialization, content-addressing, and merkle linking add overhead to every event. This is an intentional trade-off: the system is designed for audit trails that must survive years and skeptical third parties, not for maximum throughput.

### Redb (embedded) over SQLite
The journal uses `redb` with `postcard` serialization rather than SQLite. This is an explicit choice for embedded, zero-config persistence with compact binary encoding. The trade-off: no SQL query capability, but simpler deployment and no schema migrations.

### Deny-by-default over permissive-default
The default `DenyAll` mode means the agent can do nothing without explicit user configuration. This is the correct security posture for an evidence system, but creates more friction for casual use compared to permissive-default tools like Claude Code.

### One session = one journal store
Each session is self-contained in the journal — there's no cross-session query or global index. This simplifies the evidence model (each session is independently verifiable) but limits cross-session analytics (addressed by `akmon diff` and `akmon replay` operating on specific session pairs).

### No embedding dependency for core functionality
The semantic index is behind a `semantic-index` feature flag because `fastembed` pulls in ONNX runtime. This keeps the core binary lighter for users who only need the evidence pipeline.

### TUI is a consumer, not a dependency
The terminal UI (`akmon-tui`) is a separate crate from the session loop (`akmon-query`). The session communicates via channels (`mpsc`), so the TUI is one possible consumer — a headless CLI, a web UI, or an automated harness could consume the same event stream.

## Comparison to Related Projects

### vs Claude Code
Akmon is comparable in capability (tools, providers, TUI) but its differentiating feature is the evidence layer. Claude Code has no tamper-evident bundle format, no offline verification, and no cryptographically signed session records. Akmon's agent exists primarily as a reference producer for the AGEF format.

### vs Zeroclaw
Zeroclaw (31k stars) is another Rust agent runtime with OS-level sandboxing and 30+ channels. It's more focused on the agent/runtime architecture (trait-based, channel diversity). Akmon's focus is the evidence layer — the agent is a vehicle, not the destination.

### vs Microsoft Agent Governance Toolkit
Microsoft's toolkit uses hash chains + HMAC for tamper-evidence (no asymmetric signatures), has no standalone verifier, and is tied to Azure. Akmon uses Ed25519 asymmetric signatures, has a standalone `agef-verify` binary, and is cloud-independent. Akmon complements rather than replaces Microsoft's stack — it can seal what Purview captures and verify what Foundry traces.

### vs State System
State System (Python) uses model-mediated state with append-only journals. Akmon uses deterministic content-addressing at the byte level (canonical CBOR), not model interpretation. Where State System asks "what does the model think happened?", Akmon asks "what are the exact bytes, and can you verify them?"

### vs claude-replay
`claude-replay` creates HTML replays from agent sessions. Akmon's replay is deterministic re-execution — it doesn't just visualize what happened, it **re-runs the session** with playback providers/tools and compares every event. This is a much stronger verification guarantee.

## License
Apache-2.0

## Dependencies
- Rust 1.88+ (edition 2024)
- Key crates: tokio, redb, postcard, ciborium, serde, sha2, blake3, ratatui, crossterm, fastembed (optional), dunce, ignore
