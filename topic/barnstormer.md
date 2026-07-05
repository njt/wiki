---
url: https://github.com/2389-research/barnstormer
title: barnstormer
date_fetched: 2026-05-31
---

barnstormer is 2389 Research's agentic spec builder — a local-first Rust application where an AI agent swarm builds living product specifications as kanban cards. Unlike tools that treat specs as documents you write before coding, barnstormer treats the spec as the primary artifact: agents interrogate you one question at a time, populate a kanban board with decisions/constraints/ideas, and continuously emit portable artifacts (Markdown, YAML, and a DOT pipeline diagram). ~31,000 lines of Rust across 6 crates. The spec IS the output, not a precursor to code.

## Architecture

barnstormer uses **event sourcing with the actor model**. Every mutation to a spec flows through a single path:

```
Command → SpecActor → Event → SpecState (in-memory)
                       |
                       +→ JSONL log (source of truth)
                       +→ SQLite index (queryable cache)
                       +→ SSE broadcast (real-time UI)
                       +→ Exports (spec.md/yaml/pipeline.dot)
```

The `SpecActor` processes commands sequentially on a tokio `mpsc` channel. It reads state under `RwLock`, validates the command, produces events, applies them to state, broadcasts them, and persists to JSONL. This means no locks, no race conditions, no concurrent modification conflicts — all writes are serialized through one actor.

**Six crates** in a Cargo workspace:
- **barnstormer-core** — Domain types, 22 command/event variants, state reducer with undo stack, exporters (Markdown/YAML/DOT)
- **barnstormer-store** — JSONL event log (append-only, truncate-on-corruption), SQLite index (rebuildable), periodic snapshots, crash recovery
- **barnstormer-agent** — Swarm orchestrator (4–5 agents per spec), 9 domain tools, LLM provider adapters (Anthropic/OpenAI/Gemini), import via LLM parsing
- **barnstormer-server** — Axum HTTP API, SSE streaming, HTMX+Askama web UI (30+ template partials), auth middleware
- **barnstormer-runtime** — Server lifecycle, config
- **barnstormer-tauri** — Optional desktop wrapper

**Agent swarm per spec**: Manager (coordinator, only agent that streams text, handles human messages), Brainstormer (generates idea cards), Planner (organizes into structured plans), DotGenerator (narrates diagram insights — the diagram is auto-generated from cards). Each agent runs as a **mux SubAgent** with a fresh instance per reasoning step (max 10 iterations).

**Phase-gated lifecycle**: Brainstorming (only Manager runs, structured Q&A) → Refining (all agents run, cards polished) → Complete (read-only). Phase transitions happen via user confirmation of `propose_transition` questions.

**Web UI**: Server-rendered HTML via Askama templates, HTMX for partial updates, SSE for real-time event streaming. No JavaScript build step.

Key files:
- `crates/barnstormer-core/src/actor.rs` (1683 lines) — The SpecActor: validates commands, produces events, broadcasts
- `crates/barnstormer-agent/src/swarm.rs` (2322 lines) — Swarm orchestrator: agent scheduling, phase gating, question enforcement, human message prioritization
- `crates/barnstormer-core/src/state.rs` (1590 lines) — Event reducer with undo stack, context attachment tracking
- `crates/barnstormer-server/src/web/mod.rs` (8158 lines) — All web route handlers

## Key Techniques

### Agents use structured tools, not chat

Agents MUST follow a 4-step workflow enforced by the system prompt:
1. `read_state` — Get current spec as JSON
2. `emit_narration` — Explain what you're about to do (visible to user, NOT a chat)
3. `write_commands` — Submit mutations wrapped in `{"commands": [...]}`
4. `emit_diff_summary` — Mark step finished with a change summary

Three question tools (`ask_user_boolean/multiple_choice/freeform`) enforce single-pending: exactly one question at a time per spec. No agent can ask while one is pending.

### Undo via inverse event replay

Rather than tracking command history, the system captures inverse events at mutation time. `CardCreated` pushes inverse `CardDeleted { card_id }`. `CardUpdated` captures old values before mutating. Undo produces an `UndoApplied` event containing inverse events, applied via `apply_without_undo()` so the undo doesn't itself push further undo entries. Non-undoable events: canvas updates (agents regenerate), phase transitions (lifecycle), streaming deltas (ephemeral).

### Ephemeral events for streaming

`StreamingDelta` and `StreamingToolActivity` get `event_id = 0` and are never persisted. The actor skips incrementing the monotonic counter, so the JSONL log has no gaps. These events flow through the broadcast channel to SSE subscribers but leave no trace in durable storage.

### Separate broadcast receivers prevent event theft

The swarm's run loop maintains two subscribers on the broadcast channel: `phase_rx` detects `QuestionAnswered` events matching pending transitions, `wake_rx` only interrupts the idle sleep. They MUST be separate because `recv()` consumes events — sharing would silently drop transition answers. The regression test explicitly validates this.

### Context file provenance validation

Cards synthesized from uploaded files carry `source_attachment_id`. The actor **validates** this: rejecting cards that claim non-existent or removed attachments. This prevents the Manager from hallucinating provenance links. Context files are summarized asynchronously via LLM, with summary failure tracking (`summary_error` field) and a `retrieve_context` tool that answers agent questions about full file contents.

### Prompt caching saves ~25–35%

System prompts are stable within a phase, so they're marked cacheable via mux-rs `SystemBlock::cached()`. Tool definitions also cached. Per CLAUDE.md, this saves "~25-35% on Manager Sonnet input across measured workloads."

### LLM-powered import

`barnstormer import <file>` sends arbitrary text to an LLM with a structured JSON output format to extract spec metadata and cards. The result is converted to `Command` variants and fed through the SpecActor. Format hints (file extension or `--format`) help the LLM understand the input structure.

## Design Decisions

**Event sourcing for a single-user desktop app.** This is overhead for a local tool, but it buys: crash safety (truncate partial JSONL lines), complete audit trail, undo via inverse events, replay-based recovery, deterministic state reconstruction. The trade-off: more code, more disk I/O, more startup latency (replay + snapshot).

**Single question constraint.** Only one pending question per spec. Simplifies UI (one widget), forces sequential thinking. The trade-off: agents can't explore in parallel — the Manager can't ask about architecture while the Brainstormer asks about UX.

**Round-robin agent scheduling, not priority queue.** Fixed order: Manager → Brainstormer → Planner → DotGenerator. Manager gets priority after human messages via `Notify`, but otherwise no dynamic scheduling. Simple, predictable, easy to reason about.

**HTMX over JS framework.** No client-side build step, no state management, simpler deployment. The cost: complex real-time UI requires careful SSE event wiring — a known footgun documented extensively in CLAUDE.md (events vanish without matching `hx-trigger` attributes).

**Phase gating eliminates the Critic by default.** The Critic role exists in the codebase but isn't spawned in the default swarm. During Brainstorming, only the Manager runs. This means the Critic never sees raw user input before the Manager has interpreted it.

**Cards are deliberately minimal.** No tags, estimates, assignees, due dates. The spec explicitly calls advanced agile metadata a non-goal. Cards have: type, title, body (markdown), lane, order (float for stable sorting), refs, timestamps, creator, and optional source attachment link.

## Comparison Notes

**vs. other kanban agent interfaces** ([[Managing Agents via Kanban Boards]], [[ralph-ban]], [[vibe-kanban]]): Those use kanban as a task management layer for coding agents. barnstormer uses kanban as the spec authoring medium itself — the board IS the deliverable. Cards flow Ideas → Plan → Spec lanes as the spec matures.

**vs. [[Gas Town's Agent Patterns]]**: Both use hierarchical roles (Manager → workers). Gas Town's sessions are disposable; barnstormer's agents persist context (rolling summaries, key decisions, `last_event_seen` cursor) and survive restarts via snapshots.

**vs. [[ctx – Agentic Development Environment]]**: Both local-first multi-agent tools, but ctx orchestrates coding agents with worktree isolation; barnstormer orchestrates spec-authoring agents with event sourcing. ctx's concurrency model is filesystem isolation; barnstormer's is the actor model.

**vs. [[Simmer Skill]] and [[recursive-mode]]**: All three are from 2389 Research, all use phase-gated workflows and structured agent interaction. The shared philosophy: agents follow a defined process, not freeform exploration.

**vs. [[Process-Based Concurrency BEAM OTP]]**: The actor-per-spec pattern is essentially BEAM/OTP implemented in Rust with tokio channels. Simpler (no supervision tree, no hot code reloading) but same fundamental pattern.

**vs. [[Specifications as the Product]]**: barnstormer operationalizes the thesis. The spec continuously evolves, artifacts auto-generate on every change, and the DOT pipeline diagram is described as the "PRIMARY build artifact." The spec IS the output.

## Tags

#tool #project #agents #specifications #kanban #event-sourcing #rust #local-first

## Related

- [[Agent Orchestration]]
- [[Managing Agents via Kanban Boards]]
- [[Specifications as the Product]]
- [[Agent Memory and Context]]
- [[Gas Town's Agent Patterns]]
- [[ctx – Agentic Development Environment]]
- [[Simmer Skill]]
- [[recursive-mode]]
- [[2389 Plugin Marketplace]]
- [[Compound Engineering]]
- [[Process-Based Concurrency BEAM OTP]]

---

Source: [github.com/2389-research/barnstormer](https://github.com/2389-research/barnstormer) — full repo analysis from shallow clone, 2026-05-31. MIT license. ~31K lines Rust.
