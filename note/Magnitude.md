# Magnitude

Magnitude is an open-source, local-first AI coding agent: an Apache 2.0 terminal agent (with web and Electron desktop clients) that profiles your hardware, downloads a recommended model, and runs it in-process — no Ollama, no API keys, no cloud. What makes it worth studying isn't the "runs offline" pitch, which [[OpenMonoAgent]] and others also make; it's that Magnitude rebuilt the *entire* agent substrate — state, context, tooling, and the inference engine itself — around event sourcing and a native Rust inference node, rather than bolting local models onto an imperative chat loop.

---

## Architecture

Magnitude is a Bun + Turborepo monorepo, Effect-TS native (Effect 3.21, services as `Context.Tag`, Effect Schema for every serializable type). The package layering is strict, documented in `AGENTS.md`:

```text
clients (cli/web/desktop) → client-common → sdk → ACN (daemon) → agent → event-core
```

- **Clients** — `cli/` (Ink/React terminal UI), `web/`, `desktop/` (Electron). They import only from `client-common` and `sdk`, never from the daemon internals. All three share state behavior through `packages/client-common`.
- **ACN** (`packages/acn`) — the local daemon hosting the agent runtime, sessions, file ops, and display streams. It is the composition root for the agent; clients reach it over typed RPCs defined in `packages/acn-protocol`.
- **ICN** (`inference/`) — the **Inference Control Node**, a Rust workspace (`icn-contracts`, `icn-models`, `icn-hardware`, `icn-reasoning`, `icn-engine`, `icn-api`, `icn-server`) built on a pinned fork of `llama-cpp-rs`/llama.cpp. `icn-server` exposes an OpenAI-compatible HTTP/OpenAPI surface.
- **agent** (`packages/agent`) — the runtime: `coding-agent.ts` wires a set of projections and workers together; `roles/` defines the worker roles; `harness/` the turn loop; `tools/` the tool universe.
- **event-core** (`packages/event-core`) — the event-sourcing engine everything else sits on.

There is no resident coordinator process. Clients obtain one canonical ACN per data root via **JIT ensurance** (`design/acn/lifecycle/jit-spawning.md`): a single SQLite owner row (`AcnOwnerStore`) is the one durable coordination fact, replaced by compare-and-swap only after the predecessor's process group is proven absent. Launch/spawn/upgrade policy lives in six narrow authorities (`AcnConvergenceDecider`, `AcnDaemonShutdownSupervisor`, etc.), each a declared FSM, bounded by fixed timeouts (2s publication grace, 30s stall window, 5min absolute startup ceiling).

## Key Techniques

**Event sourcing as the agent's state model.** The event log is authoritative; every durable fact — context window, turn lifecycle, task graph, compaction state, conversation, display timeline, toolkit — is a *projection*, a replayable query over committed events. Workers own effects but never truth; their outcomes return as new committed events (`packages/agent/src/projections/` lists ~20 projections). Restart replays the log (or a snapshot + suffix), so sessions are resumable and auditable by construction. This is the sharpest contrast with every other coding agent, which mutate in-memory state imperatively.

**Addressed projection state.** Projections keep small indexes in memory/snapshots and push large collection bodies into addressable entries (`info/event-core/addressed-state.md`). Residency is *exactly the pinned set* — there is no cache policy, capacity, or eviction schedule. Pins come from accepted display-view windows and active producers (streaming assistant messages, tools). Structural rewrites allocate new physical entries; old ones are retained so frozen scrollback windows stay valid. A mutation's cost is proportional to the affected suffix, never the collection's total size.

**Display views page by reshaping.** Clients subscribe to windowed views of a session's display state and page by sending a *larger tail window*, not by slicing locally (`info/agent/display-views.md`). The ACN relays full state on open/shape-change and JSON patches against the last-sent state otherwise.

**Compaction as a convergence process.** Context reduction isn't a one-shot summarize call (`design/harness/compaction.md`). A soft token cap with a 16,384-token workspace (8,192 variable input + 8,192 output reserve) drives a sequential loop: each step compacts the oldest useful prefix into a summary bounded at 4,096 tokens, and if summarization fails after bounded retries, **lossy elision** (drop a prefix, leave a marker) is the deterministic liveness fallback. The guarantee: compaction eventually succeeds while a compatible model remains, and a context-rejected turn waits and resumes only after it does.

**Single-executor inference with exact-prefix reuse.** The ICN's scheduler multiplexes requests through one persistent llama.cpp context (`design/inference/engine.md`, `scheduler.md`): decode-first batch construction (one decode token per runnable sequence, then rotating prompt quanta) protects latency. Admission reuses the available sequence with the longest *exact* committed text/media prefix; a media span is indivisible and matched by stable content identity. Speculative decoding (MTP/DFlash/DSpark) tracks target-native and draft-logical positions as one linked boundary.

**Template-probing reasoning detection.** Rather than trusting model filenames, `icn-reasoning` loads a model in no-allocation mode and *differentially renders* its chat template with nonces across probe conversations to discover how reasoning is controlled, then normalizes it to a stable vocabulary — `none/minimal/low/medium/high/xhigh/max/adaptive` — bound to the template fingerprint (`design/icn/reasoning-detection.md`). Qwen's `enable_thinking`, DeepSeek's chat/thinking mode, and MiniMax M3's `adaptive` all collapse to one caller-facing scale.

**Hardware-derived memory accounting.** `MemoryTopology` is the sole authority translating a native allocation's location into a physical memory domain; native reports supply only evidence (`info/icn/memory-abstractions.md`). It fails closed on unknown/contradictory device identity rather than inferring shared memory from platform or backend — a genuinely careful design most inference tools hand-wave.

**Parity testing as engineering practice.** Three complementary suites (`inference/README.md`): correctness parity (smallest observable ops), performance parity (only after proving equivalent work), and composite benchmarking against pinned `llama-server` — with a content-addressed model registry, a native C++ oracle, and JSON evidence schemas.

## Design Decisions

**TypeScript (Effect) for the agent, Rust only for the hot path.** Opposite of [[Grok Build]] (pure Rust) and [[OpenMonoAgent]] (pure C#). Magnitude bets developer velocity and the npm/skills ecosystem matter for the *harness*, while native performance matters only inside the inference node. The cost is two toolchains and a code-generated protocol boundary between them (`icn:generate` regenerates the ICN client from the Rust OpenAPI export).

**Event sourcing buys resumability and auditability, and pays in complexity.** Every UI state change is a projection update over a log append; simple mutations become event → projection → worker → event round-trips. The payoff is a session you can `bun session`-inspect, replay point-in-time, and snapshot — impossible in imperative harnesses.

**Local-first as an ownership claim, not a feature.** Magnitude owns the whole stack: model catalog, hardware profiling, fit estimation, download, inference, agent runtime. That's the "no Ollama" promise made real — and a much heavier engineering lift than OpenMonoAgent's single embedded llama.cpp. The FAQ's positioning ("Ollama runs models, Hermes is an agent that uses them; Magnitude does both") is a declaration of scope, not just marketing.

**Explicit ownership discipline everywhere.** `design/architecture/query-mutation-state.md` and `operation-ownership.md` encode a philosophy — "one fact → one owner", mutation vs. query strictly separated, removal is finite (TERM → KILL escalation), telemetry can never authorize. This is a reaction against the ad-hoc state that plagues agent frameworks, and it reads like the project's actual thesis.

## Comparison Notes

- **vs. [[Grok Build]]** — both are open-source coding agents with serious native engineering and a formal protocol layer. Grok is a pure-Rust monolith-as-publication targeting the xAI cloud; Magnitude is TypeScript+Rust, local-first, and event-sourced where Grok uses actor-based chat state.
- **vs. [[OpenMonoAgent]]** — the same "infrastructure you own" thesis, but Magnitude replaces a fixed Qwen default with a hardware-profiled catalog, speculative decoding, prefix reuse, and a multi-agent role runtime. OpenMonoAgent optimizes for "one binary that just works"; Magnitude for an owned inference stack you can swap models across.
- **vs. [[Apache Burr]]** — Burr makes agent state an explicit developer-authored state machine; Magnitude's event-core makes it event-sourced projections with replay. Both are anti-imperative reactions to [[Agent Memory and Context]]'s "context engineering, not prompt engineering", but Magnitude is the more radical substrate.
- **vs. [[Oh My Pi (omp)]]** — both mix TypeScript/Bun with Rust, but omp inherits Pi's imperative in-process core; Magnitude's distinguishing move is the durable event log and the daemon/engine split.
- **vs. Claude Code** — closed and cloud; Magnitude is the open, offline answer with a more rigorous runtime, trading frontier-model access for local-model sovereignty (the [[Local Models in Mid-2026]] bet).

The through-line: Magnitude is the first open-source coding agent to treat **the runtime's state model** and **the inference engine** as two halves of the same first-principles project, rather than a prompt loop with a model socket. Whether that rigor survives contact with actual coding tasks — and whether local models close the gap — is the open question.

---
*Sources: [[raw/magnitude]], [[summary/magnitude]]*
*Last updated: 2026-08-21*
*Tags: #tool #project #agents #coding-agent #inference #event-sourcing #local-model #typescript #rust #multi-agent*
