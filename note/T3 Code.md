# T3 Code

T3 Code (pingdotgg) is an open-source "agent harness control surface": a local server that drives your existing coding agents — Claude Code, Codex, Cursor, Grok Build, OpenCode, Antigravity, fx — from mobile, web, and desktop clients. It matters because it is the most fully-engineered attempt yet to treat *running agents* as a distributed-systems problem: event-sourced orchestration, per-provider adapters, multi-route remote access with DPoP-authenticated sessions, and server-owned thread settlement, all in a project that still describes itself as "very very early."

---

## Architecture

- **Server owns the workspace; clients control it.** Provider processes, terminals, Git, and project files live on the server (`apps/server/src/`); web, desktop renderer, and mobile share one connection runtime (`packages/client-runtime/src/connection/supervisor.ts`) so reconnect and multi-environment behaviour can't diverge between platforms. The RPC contract (`packages/contracts/src/rpc.ts`, ~2.3k lines of Effect Schema/Rpc definitions) is the versioned boundary.
- **Event-sourced orchestration-v2** (`apps/server/src/orchestration-v2/`, ~175 files). The `Orchestrator.ts` (14,673 lines) serialises commands and decides domain events with no provider or filesystem I/O. `EventSink.ts` commits events, projections, the command receipt, and outbox effects in one transaction; `EffectWorker.ts` does side effects after intent is durably recorded, making command retries idempotent.
- **Pruned event log.** Only ~1 day of history is kept — enough for reconnect replay and retry deduplication — with `ProjectionMaintenance.ts` deleting events the projections already cover. Projections are the source of truth; the log is deliberately not rebuildable.
- **Provider adapters** (`packages/provider-*`: codex-app-server, acp, cursor, grok, opencode, pi, fx, kiro, muse…). A driver kind vs. instance distinction isolates accounts: two accounts on the same driver never share session state. Normalized commands/events mean orchestration never branches on provider identity.
- **Multi-route remote access.** Direct LAN, Tailscale, SSH, or the hosted "T3 Connect" relay (Clerk identity, Cloudflare tunnels, Durable Objects that hold webhooks while an environment is offline). The route changes nothing about the execution model; the connection supervisor probes routes with public-descriptor checks and preflights better routes while connected.
- **Native Rust helpers** (`native/`: resource-monitor, libghostty-vt, snapshot tools) beside the TypeScript monorepo, built with the team's own "vite-plus" toolchain.

## Key techniques

- **CQRS-with-receipts as agent control.** A command acknowledgement means *intent committed*, not provider work finished — the same distinction infrastructure engineers make for writes vs. effects, applied to agent turns.
- **Hidden Git refs for checkpoints** (`checkpointing/CheckpointStore.ts`): workspace snapshots never add commits to the user's branch, and a revert that a provider's conversation can't roll back is rejected *before* touching the filesystem.
- **Server-owned settlement.** Thread completion, PR discovery, and auto-settle (`ThreadSettlementService.ts`, `PullRequestSyncReactor.ts`) run without a connected client; clients render persisted state rather than deriving it from their own clocks.
- **Socket-level liveness forensics.** The connection supervisor distinguishes "OS says offline" from "socket actually dead" by asking the host for its public descriptor over a fresh request — with route-specific probe deadlines (3s loopback, 15s over DERP/VPN) — because loopback "offline" reports are often lies.
- **Model-driven evals of the tool surface itself** (`evals/`): paid-model evals grade whether agents use T3's MCP tools correctly, whether thread titles name the right subject, and whether PR descriptions survive project instructions that try to redirect them.
- **Per-provider permission containment**: e.g. on OpenCode 2.x, each thread gets its own `t3-code-<thread>` MCP entry and session rules deny every other thread's; automatic full-access grants use `once` so they can't widen a shared server's permissions.

## Design decisions

- **Build a control plane, not a harness.** Unlike projects that wrap models with their own agent loop, T3 delegates the loop to the CLIs the user already pays for and invests everything in supervision, remote access, and durability. This sacrifices harness-level control (no custom prompts/tools of its own) for provider breadth and subscription reuse.
- **Event log as cache, not archive.** Pruning the log and keeping projections authoritative trades forensic rebuildability for storage simplicity — a defensible bet for a UI control surface, but it means replay-dependent debugging is capped at a day.
- **Correctness over velocity in the core loop.** Docs obsess over idempotency, pruning-vs-replay races ("resumes recheck that sequence after reading the replay"), and Electron pre-ready platform setup. This is unusual rigor for a pre-1.0 consumer product.
- **Fork-friendliness as a product promise**: fully open source, explicitly inviting forks, while mostly refusing feature contributions — control over direction without sacrificing exit rights.

## Comparison notes

- [[happy]] and [[claude-ctrl]] are earlier, thinner takes on the same job — remote-controlling Claude Code from a phone. T3 Code generalises across seven providers and backs it with an event-sourced server rather than a bridge.
- [[vibe-kanban]] also orchestrates multiple coding agents from a control surface, but stays work-queue-shaped; T3's orchestrator is a full durable event system with per-provider adapters.
- [[Harness Engineering (OpenAI)]] and [[Everyharness]] study the harness itself; T3 Code's bet is the interesting counterpoint — that the harness is a commodity and the *supervision layer* is where product value accrues.
- Its server-owned settlement and replayable projections echo the event-sourcing debates around [[The Log — Unifying Abstraction for Real-Time Data]]: here the log is explicitly *not* the source of truth.

#tool #project #agents #orchestration #event-sourcing

---
*Sources: [[raw/t3code]], [[summary/t3code]]*
*Last updated: 2026-11-09*
