# Mecatl

Mecatl is Stacklok's open-source (Apache-2.0), cloud-native **agent harness**: a Go system that provides the agent loop, tools, permissions, hooks, delegation, and service boundaries for running AI agents as *production workloads* on infrastructure you operate, rather than as a terminal session. Its thesis is that a replaceable process needs durable external state, an append-only event record, and single-writer coordination — and that all three belong behind seams, not inside the loop.

---

## Architecture

The repo is a Go multi-module workspace (`go.work`) with a clean layered core:

- **`engine/`** — the agent loop, deliberately importable and embeddable. `engine/agent/loop.go` (~3,400 lines) holds `Engine.Run`/`drive`/`runLoop`/`dispatchTurn`; `engine/agent/compaction.go`, `subagent.go` (~4,700 lines), `teamsupervisor.go` (~2,400 lines) hold the multi-agent machinery.
- **`engine/port/`** — the seams everything plugs into: `LLMProvider`, `SessionStore`, `SessionLease`, `PermissionPolicy`, `HookRunner`, `EventLog`, `Compactor`, `Schedule`, `ToolResultRoute`. Ports use typed sentinel errors (`ErrSessionNotFound`, `ErrLeaseHeld` vs `ErrLeaseUnsupported`) so consumers can distinguish "no such session" from "backend broken" from "backend will never lease".
- **`engine/adapter/`** — reference implementations of every port: in-memory stores (`memstore`, `memfs`, `memlease`, `memledger`), plus *conformance suites* (`storeconformance`, `leaseconformance`, `eventlogconformance`, `fsconformance`…) that any third-party adapter must pass. This is the JDBC-driver trick applied to agent runtimes.
- **`engine/tool/`** — the tool contract (`Tool` interface with `Spec()`, `ReadOnly()`, `Execute(Environment)`), plus `catalog.go`, `toolsearch.go` (progressive disclosure of tools), and the `Workspace`/`FileSystem`/`Environment` seams — placed in `tool` rather than `port` to break an import cycle documented right in the file headers.
- **Binaries** — `mecated` (server, gRPC + HTTP/SSE), `mecatui` (Bubble Tea terminal client that can host a local server or attach remotely), `mecatequi` (one prompt → patch → machine-readable result), `mecak8s` (Kubernetes runtime), `mecademo` (offline scripted demo, no API key), and `mecatl-executor`/`mecatl-execution-provider`.
- **`contracts/`** — protobuf-defined gRPC API; `sdk/` is a TypeScript SDK for Node/Bun/browser clients.

The `mecak8s` runtime composes the reference deployment: Redis for session state and event logs, Kubernetes leases for one-writer-per-session, a drain path for pod replacement, and disposable replicas that recover persisted work after process replacement. An opt-in **local microVM backend** (Linux amd64, `mecated microvm doctor`) runs filesystem and shell tools inside a VM while keeping model providers and credentials on the host, with operator-only egress policy (`deny-all`/`allowlist` via `mecated serve` flags — clients and project config cannot weaken it).

## Key techniques

- **Provider-neutral DTO with a reflection tripwire.** `port.LLMRequest` is deliberately bare: the model is an opaque string, provider-private knobs (thinking budgets, store flags) are adapter constructor options, not request fields. `llm_neutral_test.go` fails the build if anyone adds a provider-branching field. The loop itself never branches on provider.
- **Reasoning replay is opaque, structurally neutral.** `ChunkReasoning` (display-only summary) is split from `ChunkReasoningItem` (the provider's opaque replay blob — OpenAI's encrypted reasoning items, Anthropic's (thinking, signature) pair). The loop stores the blob on the message and sends it back verbatim; only the *structure* is neutral. Same pattern for OpenAI's phase markers, which GPT-5.x needs on stateless replay or it treats preambles as final answers. This is hard-won multi-provider engineering most harnesses get wrong.
- **Read-parallel / mutate-serial dispatch with a per-call escape hatch.** Tools declare `ReadOnly()`; the dispatcher batches reads concurrently and serialises mutations. The subtle bit: `parentMutatingCaller` lets a nominally read-only tool (Subagent, Parallel) declare that *this particular call* will merge into the parent workspace at run end, so it flushes alone — preventing torn reads against sibling Read/Grep calls. A cross-run `SerializingMerger` mutex complements it for concurrent sessions targeting the same workspace.
- **Compaction as a seam with a safety sentinel.** `ErrCompactionWouldOrphan` — if the only producible history would orphan a tool result or dangle a tool call (which would brick the session with an HTTP 400), the compactor returns the *original* history alongside the error. The default heuristic compactor keeps the last 6 messages verbatim but "back-snaps" the cut to recent *user* turns (Codex/gemini-cli prior art): by message count the tail is all tool output, so the naive cut summarises away the live task — the exact bug "I don't have the original task… please resend" describes. A tiered cascade compactor (`cascade.go`) and a resolved-per-use context-window precedence chain (override → operator config → live metadata → models.dev catalog → 128K fallback, with typed `context_window_unavailable` gRPC rejections) sit on top.
- **Leases with fencing tokens.** `Lease.Token` is a monotonic epoch that advances only on takeover; the plumbing exists for future CAS-save stale-writer rejection (v1 enforcement is the grant itself). `ErrLeaseUnsupported` degrades *stickily* to a byte-identical no-lease path rather than retrying.
- **Event-sourced sessions.** `SessionStore.Load` may either deserialize a snapshot (`engine/adapter/sessnap`) or fold an append-only event stream (`engine/adapter/eventsource.Fold`) — the same session domain supports both systems of record.
- **Conformance-tested adapters.** Every port ships a conformance suite in `engine/adapter/*conformance`, so a new store or lease backend is validated against the loop's actual expectations, not vibes.
- **Guardrails in the loop**: `guardrailcheck.go`, `leakgate`, `fence.go` (fence forgery tests), secret-scrubbed command environments, deny-dominant permissions with approval park/resume (`parkAuthorization`, `ResumeApproval`), and delegated capabilities that can only narrow across in-process hops.

## Design decisions

The trade the project makes is **seam completeness over minimalism**. Where a minimal harness hardcodes a model client and an in-process history slice, Mecatl pays for a port layer, conformance suites, typed sentinel errors, and sticky-degradation contracts — because the target is Kubernetes replicas sharing sessions, not a laptop. The cost is visible: ~12,000 lines in `engine/agent` alone, and architecture docs that read like database-engine specs (2,193 lines, frozen ADRs, a generated formal domain model). The reward is that embedding the engine (`engine/COMPATIBILITY.md` compatibility contract) or swapping Redis for anything else is a supported act, not surgery.

Honesty is also a design decision: the README states plainly that `mecated` is unauthenticated by default (loopback/single-user only until hardened), and that cross-process cryptographic proof and authority attenuation are *active design work* with a public identity-model doc and tracker issue — not shipped guarantees dressed as features.

A meta-note: the repo develops itself with Claude Code — `.claude/agents/` ships a dozen reviewer subagents (secure-code-reviewer, devils-advocate, kubernetes-operator-expert…) and `.claude/skills/` ships skills like `plan-orchestrate` and `panel-review`. The harness's own quality gates are agent-run.

## Comparison notes

- Versus terminal-first harnesses like Claude Code or Codex, Mecatl inverts the center of gravity: the loop is a library and the server is the product, with the TUI demoted to one client. It is closer to [[The Case Against Building Your Own Agent Platform]]'s subject matter — but it *is* the platform, arguing that seams are what make a harness embeddable rather than a monolith to rebuild.
- Its read-parallel/mutate-serial dispatch and parent-merge serialization directly address the shared-workspace races described in [[Parallel Coding Agents Guide]], but as a dispatch invariant in the loop rather than a user-level orchestration habit.
- Its compaction-with-verbatim-recent-user-turns answers the failure mode behind [[Self-Generated Prompt Injections in Compaction Summaries]] and the "lost the task" drift in [[Context Rot]] — and its refusal to emit tool-pairing-invalid histories is a correctness guard most summarizers lack.
- The Redis + lease + event-log runtime is the concrete production counterpart to [[Headlong — A Microharness for Persistent Agents]]: Headlong sketches persistence for a personal harness; Mecatl engineers it with fencing tokens, drain paths, and conformance suites. It also supplies the loop-side machinery the fleet-level essays ([[Multi-Agent Systems Have a Distributed Systems Problem]], [[Designing Agent-First Platforms (Microsoft Foundry)]]) assume exists somewhere.
- The microVM egress policy (host-only flag, allowlist enforced at startup, clients cannot weaken it) is a concrete implementation of the sandbox posture in [[A Deep Dive on Agent Sandboxes]] and [[Fences Not Sandboxes]].

#tool #project #agents #harness #kubernetes #sandboxing

---
*Sources: [[raw/mecatl]], [[summary/mecatl]]*
*Last updated: 2026-10-02*
