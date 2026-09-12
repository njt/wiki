---
url: https://github.com/2389-research/barnstormer
title: "barnstormer: Agentic Spec Builder"
author: 2389 Research, Inc.
date_fetched: 2026-05-31
date_published: 2026-02-10
topics:
  - specifications-as-the-product
  - coding-agents-and-frameworks
---

# barnstormer — Agentic Spec Builder

A local-first, web-based, agentic product-spec workstation built in Rust by 2389 Research. An AI agent swarm interrogates the user one question at a time, continuously builds a living spec as kanban cards + document view, and emits portable artifacts (Markdown/YAML/DOT). ~31,000 lines of Rust across a Cargo workspace of 6 crates. MIT licensed.

Archived analysis from full repo clone and code reading conducted 2026-05-31.

---

## Architecture

### Workspace Structure (6 crates)

| Crate | Lines | Purpose |
|---|---|---|
| `barnstormer-core` | ~4,000 | Domain types, event/command definitions, state reducer, actor, exporters (Markdown/YAML/DOT) |
| `barnstormer-store` | ~1,400 | JSONL event log, SQLite index, snapshots, crash recovery, storage manager |
| `barnstormer-server` | ~9,000 | Axum HTTP API, SSE streaming, Askama+HTMX web UI, auth, config |
| `barnstormer-agent` | ~4,000 | Agent runtime, LLM provider adapters, swarm orchestrator, 9 domain tools |
| `barnstormer-runtime` | ~400 | Server lifecycle management, config |
| `barnstormer-tauri` | ~800 | Desktop wrapper using Tauri |

### Core Data Flow

```
Command → SpecActor → Event → SpecState (in-memory via RwLock)
                       |
                       +→ JSONL log (durable, append-only source of truth)
                       +→ SQLite index (queryable cache, rebuildable)
                       +→ SSE broadcast (real-time to web UI via tokio broadcast)
                       +→ Exports (spec.md, spec.yaml, pipeline.dot)
```

### Event Sourcing with Actor Model

Every mutation to spec state goes through a single path:
1. Client (human via UI, or agent via tool) sends a `Command`
2. `SpecActor` processes commands sequentially on a single `mpsc` channel (buffer 64)
3. Command validation reads current state under `RwLock`, produces `EventPayload` variants
4. Events are applied to in-memory state, broadcast via `tokio::sync::broadcast` (capacity 4096), and persisted to JSONL
5. Response returned to caller via `oneshot` channel

Key files:
- `crates/barnstormer-core/src/actor.rs` (1683 lines) — The SpecActor: command processing, event generation, state application
- `crates/barnstormer-core/src/command.rs` (357 lines) — 22 command variants covering spec lifecycle
- `crates/barnstormer-core/src/event.rs` (375 lines) — 22 event payload variants, ephemeral vs durable distinction
- `crates/barnstormer-core/src/state.rs` (1590 lines) — SpecState with event reducer (`apply()`), undo stack, context attachments

**Ephemeral events**: `StreamingDelta` and `StreamingToolActivity` get `event_id = 0` and are never persisted to JSONL. This avoids gaps in the monotonic event ID sequence.

### Agent Swarm Architecture

Each spec gets a `SwarmOrchestrator` managing 4 agents (in order):
1. **Manager** — Primary coordinator, only agent that streams text, handles human messages, proposes phase transitions
2. **Brainstormer** — Generates creative idea cards during Refining phase
3. **Planner** — Organizes ideas into structured plans, moves cards between lanes
4. **DotGenerator** — Analyzes spec structure and narrates diagram insights (diagram is auto-generated from cards)

Optional: **Critic** — Reviews for gaps, inconsistencies, risks (not spawned by default).

Each agent runs as a **mux SubAgent** — a fresh one is created per reasoning step (think-act loop with max 10 iterations). The SubAgent gets:
- A role-specific system prompt with phase-awareness block
- A 9-tool registry built per-step from the spec actor handle
- Prompt caching enabled (Anthropic `cache_control` for system blocks and tool definitions)
- Streaming hook for the Manager only
- Context built from `AgentContext`: state summary, recent events, recent transcript, rolling summary, key decisions, context file attachments

Key file: `crates/barnstormer-agent/src/swarm.rs` (2322 lines)

### Phase-Gated Workflow

The spec goes through three phases:
1. **Brainstorming** — Only the Manager runs. One question at a time. No cards created until decisions are made.
2. **Refining** — All agents run. Cards are polished, reorganized, gaps filled.
3. **Complete** — Read-only. Agents answer questions but don't propose further transitions.

Phase transitions happen via user confirmation: the Manager calls `propose_transition`, which creates a pending transition question. When the user answers "yes", the swarm watches for `QuestionAnswered` events and fires `TransitionPhase`. The swarm keeps separate broadcast receivers for phase-watching and wake-signaling to avoid race conditions where `QuestionAnswered` events get consumed by the wake receiver before the phase watcher sees them.

Key detail: `drain_transition_answers()` MUST run BEFORE the Manager re-runs after a human message, or the Manager can re-propose and overwrite `pending_transition_question`, silently dropping the original answer.

### Web UI: HTMX + SSE + Askama

The web UI uses server-rendered HTML with Askama templates (30+ template partials). HTMX handles partial page updates. SSE provides real-time event streaming to update the board, activity feed, and chat panel without polling.

Key architectural constraint from CLAUDE.md: "`htmx-ext-sse@2.2.2` only subscribes to SSE event names that appear in a `hx-trigger='sse:<name>'` or `sse-swap='<name>'` attribute somewhere inside the `sse-connect` element. SSE events with no matching attribute are received by the browser's EventSource but never dispatched anywhere — they vanish." This required hidden `<span>` elements as subscription sinks.

Template structure:
- `templates/index.html` — Main page layout
- `templates/spec_page.html` — Spec view with board, document, activity panel
- `templates/partials/` — 20+ HTMX-swappable partials: board, cards, chat, transcript, context panel, phase stepper, agent status LEDs

Key file: `crates/barnstormer-server/src/web/mod.rs` (8158 lines) — All web route handlers

---

## Key Techniques

### 1. Domain tools as the agent interface (not chat)

Unlike most agent frameworks where agents interact via freeform chat, barnstormer agents MUST use structured tools:

```
read_state → emit_narration (explain plan) → write_commands (make changes) → emit_diff_summary (finish)
```

The 9 registered tools (`crates/barnstormer-agent/src/mux_tools/`):
- `read_state` — Returns condensed spec state as JSON
- `write_commands` — Submit commands wrapped in `{"commands": [...]}`
- `emit_narration` — Post to activity feed (NOT a chat expecting response)
- `emit_diff_summary` — Mark step finished with change summary
- `ask_user_boolean/multiple_choice/freeform` — Question tools with single-pending enforcement
- `propose_transition` — Request phase change (creates a pending question)
- `retrieve_context` — Fetch full text of an attachment, optionally answering a question about it

This is enforced by the tool usage guide appended to every agent's system prompt, which explicitly spells out the 4-step workflow.

### 2. Undo via inverse event replay (not command reversal)

Rather than tracking "what command caused this," the undo system captures inverse events at mutation time:

- `CardCreated` pushes inverse `CardDeleted { card_id }` onto the undo stack
- `CardUpdated` captures old values before mutation and pushes inverse update
- `ContextAttached` pushes inverse `ContextRemoved`
- `UndoApplied` events contain the inverse events, which are applied via `apply_without_undo()` (so the undo doesn't itself push undo entries)

Non-undoable events (no undo entry pushed): canvas updates (regenerated by agents), phase transitions (lifecycle), streaming deltas (ephemeral), summarization/summarize-failed (idempotent replacement).

### 3. Ephemeral events with zero IDs for streaming

Streaming deltas and tool activity are broadcast to SSE subscribers but never persisted. They get `event_id = 0` and the actor skips incrementing the monotonic counter. This avoids gaps in the JSONL event log. The `is_ephemeral()` method on `EventPayload` gates both ID assignment and persistence.

### 4. Prompt caching via mux-rs SystemBlock

Agent system prompts are stable within a phase, so they're marked cacheable:

```rust
let mut definition = AgentDefinition::new(runner.role.label(), system_prompt.clone())
    .system_block(SystemBlock::cached(system_prompt))
    .cache_tools(true)
```

This uses the (at time of writing) unreleased prompt-caching changes in mux-rs (pinned to a specific commit). The CLAUDE.md notes this saves "~25-35% on Manager Sonnet input across measured workloads."

### 5. Context file summarization pipeline

When a user uploads files during spec creation (`multipart/form-data`):
1. File is validated (UTF-8, max 20MB per part, streaming cap enforcement)
2. Written to disk under `<spec_dir>/files/`
3. Attached via `Command::AttachContext`
4. Async summarizer calls the LLM to generate a summary
5. Summary stored via `Command::SummarizeContext` (idempotent — success clears prior failure)
6. On failure, `Command::MarkContextSummarizeFailed` records the reason
7. Cards synthesized from attachments carry `source_attachment_id` for provenance

The `retrieve_context` tool implements question-mode: the agent asks a question about an attachment, and the summarizer calls an LLM with the full file content to answer it. This is implemented in the server crate but exposed through the agent crate via a trait (`AttachmentSummarizer`).

### 6. Separate broadcast receivers to prevent event theft

The swarm run loop maintains two subscribers on the actor's broadcast channel:
- `phase_rx` — Drained with `try_recv` to detect `QuestionAnswered` events matching pending transitions
- `wake_rx` — Used only to interrupt the idle `select!` sleep when any event arrives

They MUST be separate because `recv()` consumes events. Sharing one would silently drop `QuestionAnswered` events and strand transitions. The regression test `run_loop_advances_phase_when_user_confirms_transition` validates this.

### 7. Import via LLM parsing

The `barnstormer import` CLI subcommand sends arbitrary text (file, stdin, or `--text`) to an LLM with a structured JSON output format. The LLM extracts: spec title/one_liner/goal, optional extended fields, and cards with type/title/body/lane. The result is converted to `Command` variants and fed through the SpecActor. Format hints (file extension or `--format`) help the LLM understand the input.

---

## Design Decisions

### Trade-offs

**Simplicity over flexibility**: Cards have deliberately minimal metadata (type, title, body, lane, order, refs). No tags, estimates, assignees, due dates, or custom fields. The spec says: "keep cards minimal" and "tagging/filtering/advanced agile metadata" is explicitly a non-goal for v1.

**Single question constraint**: Only one pending question per spec at a time. This simplifies the UI (one widget) and forces agents to think sequentially rather than bombarding the user. The trade-off is that parallel exploration is impossible — the Manager can't ask about architecture while the Brainstormer asks about UX.

**Event sourcing at small scale**: Full event sourcing (JSONL + snapshots + SQLite) for what is essentially a single-user desktop app. This is overhead for a local tool, but it buys: crash safety, complete audit trail, undo/redo, replay-based recovery, and deterministic state reconstruction. The SPEC.md explicitly lists "crash-safe recovery" as a goal.

**Round-robin agent scheduling (not priority queue)**: Agents run in fixed order (Manager → Brainstormer → Planner → DotGenerator). The Manager gets priority after human messages via `Notify`, but otherwise it's simple round-robin. No dynamic scheduling based on urgency or backlog.

**HTMX over SPA framework**: The SPEC.md's open design choices considered "Leptos/Dioxus/Yew" but the implementation chose HTMX + server-rendered templates. This means: no JavaScript build step, no client-side state management, simpler deployment. The trade-off: complex real-time UI updates require careful SSE event wiring (as documented in the extensive SSE gotcha in CLAUDE.md).

**Phase gating eliminates the Critic by default**: During Brainstorming, only the Manager runs — the Brainstormer, Planner, DotGenerator, and Critic are all skipped. This is intentional (structured Q&A before card generation) but means the Critic never sees the raw user input before it's been interpreted by the Manager.

### Design strengths

- **Single-writer actor**: All state mutations funnel through one sequential channel. No locks, no race conditions, no concurrent modification conflicts. Agents can be as concurrent as they want — their tool calls just queue up.
- **Dual-purpose ephemeral events**: Streaming events serve the UI in real-time without polluting the durable event log. This is cleaner than separate channels.
- **Context file provenance**: Cards link back to source attachments via `source_attachment_id`. The actor validates this — rejecting cards that claim non-existent or removed attachments. This prevents the Manager from hallucinating provenance.
- **Self-healing storage**: SQLite corruption → rebuild from JSONL. JSONL corruption → truncate last partial line. The system survives crashes by design.

---

## Comparison Notes

### vs. Kanban Board Agent Interfaces ([[Managing Agents via Kanban Boards]], [[ralph-ban]], [[vibe-kanban]])

barnstormer IS a kanban board for agents, but with a crucial difference: the board IS the spec. Those projects use kanban as a task management layer for coding agents; barnstormer uses kanban as the spec authoring medium itself. Cards flow from Ideas → Plan → Spec lanes as the spec matures. The agents populate, reorganize, and refine the board, and the board becomes the deliverable.

### vs. [[Gas Town's Agent Patterns]]

Gas Town uses hierarchical roles and ephemeral sessions. barnstormer shares the hierarchical role pattern (Manager → workers) but differs fundamentally: barnstormer's agents persist context (rolling summaries, key decisions, last_event_seen cursor) and survive restarts via snapshots. Gas Town's sessions are disposable. barnstormer's event sourcing means every agent action is durable and replayable.

### vs. [[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]]

Both use file-based memory and heartbeat loops, but barnstormer's approach is more structured: the event-sourced state IS the memory, and the agent context is derived from it deterministically. Mission Control uses Convex for persistence; barnstormer uses simple files (JSONL + SQLite) in a local directory.

### vs. [[ctx – Agentic Development Environment]]

Both are local-first multi-agent orchestration tools, but ctx orchestrates coding agents with worktree isolation for parallel git branches. barnstormer orchestrates spec-authoring agents with event sourcing for a single spec. ctx's concurrency model is filesystem isolation; barnstormer's is the actor model.

### vs. [[Spec-Driven Development]] and [[Specifications as the Product]]

barnstormer operationalizes the "specs are the durable artifact" thesis. The SPEC.md explicitly says specs are "living" — they evolve continuously, and the artifacts (spec.md, spec.yaml, pipeline.dot) are auto-generated on every change. The DOT pipeline diagram is described as the "PRIMARY build artifact." This inverts the typical software project: the spec IS the output, not a precursor to code.

### vs. [[Process-Based Concurrency BEAM OTP]]

barnstormer's actor-per-spec pattern (one `SpecActor` per spec, sequential command processing, broadcast events) is essentially BEAM/OTP implemented in Rust with tokio channels. It's simpler (no supervision tree, no hot code reloading) but follows the same fundamental pattern: isolated state, message passing, crash isolation.

### vs. [[Simmer Skill]] and [[recursive-mode]]

All three are from 2389 Research and share DNA: phase-gated workflows, structured agent interaction patterns. simmer uses judge-generator loops for artifact refinement; recursive-mode uses file-backed phases with audit loops; barnstormer uses a brainstorming→refining→complete lifecycle with question-gated transitions. The pattern across all three: **agents follow a defined process, not freeform exploration**.

---

## Tags

#tool #project #agents #specifications #kanban #event-sourcing #rust #local-first

## Cross-links

- [[Agent Orchestration]] — Multi-agent coordination patterns
- [[Managing Agents via Kanban Boards]] — The kanban-as-agent-interface concept
- [[Specifications as the Product]] — Specs as durable artifacts
- [[Agent Memory and Context]] — Context management strategies
- [[Gas Town's Agent Patterns]] — Comparable agent orchestrator design
- [[ctx – Agentic Development Environment]] — Another local-first multi-agent tool
- [[Mission Control — Bhanu's 10-Agent Squad on OpenClaw]] — Multi-agent reference architecture
- [[Simmer Skill]] — Also from 2389 Research; shares judged-loop pattern
- [[recursive-mode]] — Also from 2389 Research; shares phase-gated workflow
- [[Compound Engineering]] — Adding systems instead of manual review
- [[Process-Based Concurrency BEAM OTP]] — The actor model barnstormer implements
- [[Distributed Systems]] — Agent orchestration as distributed systems
- [[2389 Plugin Marketplace]] — Same organization

---

Source: https://github.com/2389-research/barnstormer  
Fetched: 2026-05-31 (full repo analysis from shallow clone)
