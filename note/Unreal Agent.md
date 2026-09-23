# Unreal Agent

Unreal Agent (Unreal Labs, Go, ~11K hand-written lines) is an agent harness whose defining bet is that **tool calls are asynchronous operations, not blocking steps**. The model fires calls; each runs in the background; each completed result wakes a fresh LLM turn; the harness babysits running calls so the model never has to poll. Everything else in the design — the append-only session log, the pure tool translators, the actor-runtime operation manager — falls out of that one choice. It matters as a clean, small, fully testable counterpoint to the giant multi-thousand-tool single-agent harnesses, and as a worked example of event-sourcing applied to the agent control loop.

---

## Architecture

Single-process, single-agent, event-driven. The components from the README glossary map to real packages:

- **Coordinator** (`harness/coordinator/loop.go`, ~980 lines) — the control loop. A `select` over four event sources (inbox inputs, operation updates, a heartbeat timer, model responses), followed by a `processEvents` pass that *slurps* pending channel items in a batch (1ms idle timeout, 100-item cap) before reconciling tool calls. Every accepted item is written to local state and the session store in the same step, so state and durable log never diverge.
- **Inbox** (`harness/inbox/`) — session-scoped, in-memory idempotency keyed by caller-supplied globally unique IDs that survive redelivery. Inputs are external messages or control messages (StopHard, StopWhenIdle, Heartbeat, UpdateSettings).
- **Session store** (`harness/sessionstore/localfile/`) — append-only JSONL per session (`*.session.jsonl`), with versioned items, fork records, and cached write-state. Operations and their triggering tool-call status are recorded atomically.
- **Context builder** (`harness/contextbuilder/builder.go`) — stateful, pure, in-memory assembly of model input using a committed-prefix / staged-suffix split: `AddModelResponse` appends to the committed prefix, new inputs/reasoning stage until a turn commits. It has a `TurnCompaction` turn type for long-context handling, and `Build()` returns the request plus a record of anything omitted or compacted. No I/O, no persistence dependencies.
- **Tool registry + translators** (`harness/tool/`) — a deliberately tiny fixed surface: `Bash`, `ViewImage`, `SkillUse`. Translators run synchronously on the event loop, must not do I/O, and convert a tool call into a validation result plus one or more serializable **operations**. Result formatting is a separate `TranslateResult` pass over recorded status + operation outputs.
- **Operation manager** (`harness/operation/local_manager.go`) — an actor: a goroutine owning channels for adds, cancellations, primitive events, and updates. Operations (`harness/operation/operation.go`) carry a version, a JSON state blob, an idempotency blob, and status (`ready → awaiting → canceling → completed/failed/canceled`). The local implementation is swappable — the README's headline extension is a proxy manager forwarding serialized operations to a process inside a remote sandbox.
- **Primitives** (`harness/primitives/`) — the lowest layer: process spawning (with per-OS signal handling), file creation (with fsync variants for darwin/unix), timers, SSE, and remote job plumbing. Operations are compositions of primitive dispatches; `Step{Operation, Dispatches}` is the checkpoint/advance unit.
- **LLM adapters** (`harness/llm/`) — a normalized responses-API layer with five provider clients (openai, openaicodex with credential handling, openrouter with automatic prompt caching, fireworks, ollama) and transport retry policies.

The single binary, `cmd/unreal-agent-runner`, executes one JSON request and writes persisted session items as JSONL — a batch/pipe-friendly interface rather than a REPL.

## Key techniques

- **Event-sourced agent state.** Every input, turn, model response, tool-call status, and operation update is a versioned log item; `restore()` replays the history page by page (256-item pages) to rebuild coordinator state. Recovery is a first-class, fuzz-tested behavior (`fault_fuzz_test.go`, `recovery_sequences_test.go`, `fault_fakes_test.go`), which almost no agent harness does seriously.
- **The tool-call grace period.** After a model response produces new tool calls, a 1-second grace timer (`toolCallRunGracePeriod`) batches adjacent operation completions before waking the model — so results landing together arrive in one turn instead of triggering a cascade of near-simultaneous turns.
- **Heartbeats as control-plane inputs.** A configurable timer injects a Heartbeat control message (listing running call IDs as JSON) as a *user message* into the context — the model wakes and checks on long-running calls without the harness blocking on its behalf.
- **Pure-translator discipline.** Translators synchronously validate and emit operations; execution state is tracked separately and formatted back by `TranslateResult`. This is what makes replay, forking, and remote-sandbox dispatch possible without re-running validation.
- **Stop semantics as a state machine.** `StopHard` cancels the model call and all non-terminal operations and exits only when nothing is pending; `StopWhenIdle` exits only when the loop is fully idle (`isIdle()` checks model-call, inputs, tool calls, and operations).
- **Prompt-as-contract.** The embedded preamble (`contextbuilder/prompts/preamble.md`) tells the model the concurrency model explicitly: "prefer to go wider with tool calls — they are cheap," calls run in the background, a running call shows a placeholder, and "ending a turn with nothing running ends the session."

## Design decisions

- **Concurrency in the model-facing semantics, not just the runtime.** Most harnesses serialize tool calls behind one loop; Unreal pushes asynchrony into the protocol the model itself sees (placeholders, wake-on-completion, heartbeats). The trade-off: the model must reason about partially-complete call state, which weaker models will fumble.
- **Small fixed tool surface over tool sprawl.** Bash + ViewImage + skills is a bet that composition via shell beats a large curated catalog — the opposite of the "single agent with thousands of tools" pattern in [[Ramp — Lessons from Building a New AI Product]].
- **Interfaces with one local implementation.** The operation manager and session store are single-implementation interfaces today; the sandbox-proxy example shows the abstraction is the point, but it also means the remote path is asserted, not yet shipped.
- **Known rough edge.** A FIXME in `loop.go` admits forks leave inherited tool calls without results and retain pending-input accounting — fork semantics are only partially reconciled with the loop state.
- **Go, stdlib-leaning.** Three direct dependencies (oapi-codegen runtime, image, sys). No LangChain-style framework; the harness *is* the product.

## Comparison notes

- Against [[Introducing Omnigent]], which wraps *existing* agents behind a uniform API for composition, Unreal Agent builds the harness itself and makes the async operation model the API. Omnigent is about composition; Unreal is about the loop's internal discipline.
- Against [[Apache Burr]], which frames agents as explicit state machines with persistence and replay, Unreal achieves the same event-sourcing properties with a goroutine actor model rather than decorator-declared states — same destination, very different shape.
- Its session-inbox idempotency, durable operations, and remote-sandbox dispatch target the same problems [[Best Infrastructure Platforms for Coding Agents in 2026]] surveys at the platform level, but Unreal keeps execution local-and-swappable instead of selling sandboxing as the product.
- The context builder's commit/stage split and compaction turns are a narrower take on the problems [[Context Engineering at the Frontier (Linus Lee)]] discusses: no retrieval pipelines, just disciplined append-and-compact over one canonical log.

Tags: #tool #project #agents #go #event-sourcing

---
*Sources: [[raw/unreal-agent]], [[summary/unreal-agent]]*
*Last updated: 2026-09-23*
