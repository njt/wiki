# Paperclip

Paperclip is an open-source (MIT) control plane for running a *team* of AI agents as a company. Individual coding agents — Claude Code, Codex, Cursor, Gemini, OpenClaw — are the employees; Paperclip supplies everything an organization needs to make a collection of them produce work instead of collide: an org chart, goals, budgets, governance, approval gates, and a task-manager UI. The tagline — "if OpenClaw is an employee, Paperclip is the company" — is also a precise statement of the architecture: Paperclip never reimplements the agent, it *employs* it, and the entire hard problem is the organizational coordination layer on top.

---

## Architecture

**A monorepo with a Postgres spine.** The repo is a pnpm workspace: `server/` (the Node.js control plane), `ui/` (React), `cli/`, plus `packages/` holding `db` (Drizzle ORM over Postgres, 135 tables, 273 migrations), `shared`, `adapter-utils`, `paperclip-runner` (a Rust execution runtime), and one adapter package per agent provider under `packages/adapters/`. The domain model is fully relational — everything is a table, and coordination is done *in the database* rather than in a message queue.

**The agent is a row, the adapter is polymorphic.** `packages/db/src/schema/agents.ts` defines an agent as `name`, `role`, `title`, a self-referential `reportsTo` column (the org chart is literally a foreign key to the same table), an `adapterType` string plus `adapterConfig` JSON, `budgetMonthlyCents`/`spentMonthlyCents`, `permissions`, and `lastHeartbeatAt`. The README's "if it can receive a heartbeat, it's hired" is implemented as a plugin contract in `packages/adapter-utils/src/types.ts` — `ServerAdapterModule` — whose `execute(ctx) → AdapterExecutionResult` is the only thing a provider must implement. `server/src/adapters/registry.ts` registers ~17 built-in adapters and hot-loads external plugin adapters that can override a built-in type (with pause/restore fallback).

**Hexagonal layering for the hard parts.** The execution-critical modules — `server/src/modules/wake-queue`, `run-dispatch`, `active-run-watchdog` — each follow a `domain/` + `application/` + `adapters/` split. `domain/policy.ts` is the distinctive piece: pure decision functions that branch on a "facts object" and *never* query the database, read the clock, or touch the payload. The application layer reads the DB, packs facts, calls the pure decider, then writes. This is what makes a 28,000-line `server/src/services/heartbeat.ts` reviewable at all.

**Tasks carry goal ancestry and execution locks.** `packages/db/src/schema/issues.ts` shows an issue with `goalId`, `parentId` (subtask trees), `projectId`, `projectWorkspaceId`, plus `checkoutRunId` / `executionRunId` / `executionLockedAt` — the atomic checkout. Every row is `companyId`-scoped, giving multi-tenant isolation for free at the schema level.

## Key techniques

**The wake/execution-lock state machine.** This is the crown jewel. When a wake arrives (assignment, @-mention, comment, cron routine) for an issue that already has a running execution, `decideWakeAdmission` (`wake-queue/domain/policy.ts`) picks one of three outcomes: **coalesce** (merge the wake's context into the still-running run), **defer** (queue it behind the active run), or **proceed** (queue as an ordinary run). A "zombie-run filter" (`liveRunExecutions`) prevents coalescing into a run whose process actually died. When a run releases the lock, `runReleaseDrain` promotes deferred wakes one-at-a-time in FIFO order, then `decideReleaseRecovery` re-queues an automatic recovery run if the issue was stranded (failed/timed-out/cancelled), or blocks it with a notice if the recovery agent isn't invokable. All claims are compare-and-set inside one Postgres transaction — no double-work, no double-spend.

**Partial unique indexes as the self-healing primitive.** The "self-healing runs" roadmap item is implemented as partial unique indexes over `(companyId, originKind, originId)` with `WHERE status not in ('done','cancelled')` (see `issues.ts`). Each `originKind` — `routine_execution`, `harness_liveness_escalation`, `task_watchdog`, `stranded_issue_recovery`, `onboarding_first_task` — gets its own index, so the system can spawn recovery issues idempotently: the second concurrent create loses the race atomically instead of duplicating.

**A provider-neutral transcript.** Every adapter normalizes its output into a shared `TranscriptEntry` discriminated union (`adapter-utils/src/types.ts`) — tool calls, thinking, `workspace_change`, `runtime_request`, and crucially `run_result`, which carries a `disposition: done | blocked | needs_review | yielded`. The control plane doesn't parse a provider's wire format to know if work finished; it reads the normalized disposition. The process adapter (`server/src/adapters/process/execute.ts`) shows the minimal "heartbeat protocol": spawn the CLI, inject `PAPERCLIP_RUN_ID` + a per-run minted `PAPERCLIP_API_KEY`, and let the agent report back.

**Three tool-delivery strategies.** `runtimeToolDelivery` is `native_mcp` (spawn an MCP server the agent connects to — Claude/Codex), `environment` (env vars — Cursor, Gemini, bash), or `invocation_context` (passed in the invoke call — cloud/webhook bots like OpenClaw). The same run-scoped control tools (connections search/request) reach each agent through whichever channel its runtime can actually speak.

**Cost deltas, not cost claims.** `AdapterExecutionResult.usageBasis` distinguishes `per_run` from `session_cumulative` token totals, so the server can delta consecutive runs of a persisted session instead of double-counting — the accounting that makes "budget hard-stops pause agents" trustworthy.

**Compaction that knows what the harness knows.** `session-compaction.ts` ships two policies: adapters with confirmed native context management (Claude Code, Codex, Hermes) get `ADAPTER_MANAGED_SESSION_POLICY` — Paperclip *never* rotates them; others get threshold-based rotation (200 runs / 2M input tokens / 72h). This is an honest admission that re-summarizing a session the provider is already managing would corrupt it.

## Design decisions

**Optimized for atomicity and auditability over simplicity.** The wake-queue machinery is enormous because the authors chose "no double-work and no runaway spend" as a hard invariant, enforced in the database rather than in the agent. The trade-off is visible: a 28k-line heartbeat service, a 135-table schema, and a state machine that most orchestration tools approximate with a `while` loop.

**Bring-your-own-agent, explicitly.** The README's "What Paperclip is not" list is a design stance: not a chatbot, not an agent framework, not a workflow builder, not a prompt manager, not a single-agent tool. Paperclip refuses to compete on the agent layer — it orchestrates whatever CLI or bot the user already trusts, which is why it patches `@agentclientprotocol/claude-agent-acp` and `@openai/codex` rather than talking to model APIs directly.

**Postgres as the single coordination primitive.** There is no separate queue broker or leader election: the wake queue, execution locks, routine schedules, and budgets all live as rows, and atomicity comes from transactions and partial unique indexes. For a self-hosted product that must be installable with `curl | bash` and an embedded Postgres, this is the right call — one dependency, one concurrency model.

**Company as the isolation unit.** Multi-tenancy is column-level (`companyId` on every table), not deployment-level, so one instance runs many isolated companies with separate audit trails and secrets. The roadmap's unfinished items — Memory/Knowledge, work queues, self-organization, automatic organizational learning — show where the authors know the product still ends: it coordinates agents today, but does not yet give them a shared memory.

## Comparison notes

Among the meta-harnesses, Paperclip is the most *organizational* — and the least about the agent. [[QM (Multiplayer Agent Harness)]] also wraps Claude Code/Codex/Pi/OpenCode as company infrastructure, but standardizes the *tool surface* (a fixed set with `execute` as the escape hatch) and organizes around scopes (person/channel/team) with per-scope memory and keychains; Paperclip standardizes the *employment contract* — heartbeat + provider-neutral transcript + budget + org-chart position — and organizes around a literal reporting hierarchy. QM asks "what can this agent touch?"; Paperclip asks "who does this agent report to, and has it hit its budget?"

[[bb — The Agent Orchestrator as Normalizer]] is the closest engineering cousin, but one level down: bb normalizes *in-thread events* (provider events → shared `ThreadEvent[]`) at the protocol layer, with no governance or budgeting. Paperclip normalizes *run outcomes* (the `run_result` disposition) at the work-product layer, and spends most of its code on what bb leaves out — atomic checkout, recovery, budgets, approvals, audit logs.

[[Multi-Agent AI Systems Are Organizations]] argued that multi-agent systems face the four organizing problems of the Carnegie tradition by construction. Paperclip is the productization of that thesis: task division (issues + subtask trees), allocation (atomic checkout + assignee), information provision (goal ancestry flowing down every task), and reward distribution (per-agent budgets) are all first-class CRUD objects. It is also an existence proof for the paper's punchline — that these are solved by designing the *organization*, not the agent.

The [[Agent Orchestration]] hub's primitives all appear here as shipped features: atomic task claiming (the execution lock), worktree isolation (execution workspaces via git worktrees), ephemeral sessions with persistent state (session resume + task context across heartbeats), heartbeat loops, and kanban as the human interface. Paperclip's contribution is showing what happens when all of them are built into one coherent control plane with an org chart and a budget attached.

#tool #project #agents #orchestration #database #governance

---
*Sources: [[raw/paperclip]], [[summary/paperclip]]*
*Last updated: 2026-09-11*
