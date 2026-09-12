# Paperclip

Paperclip is an open-source (MIT), self-hosted control plane for running a *team* of AI agents as a company — a "zero-human company" where you hire agents into an org chart (CEO, CTO, engineers, marketers), point them at a single company goal, and govern the result as the board of directors. Individual coding agents — Claude Code, Codex, Cursor, Gemini, OpenClaw — are the employees; Paperclip supplies everything an organization needs to make a collection of them produce work instead of collide: org charts, goals, per-agent budgets, goal inheritance, tickets, approval gates, and a task-manager UI. Its distinguishing move is deliberate anti-framework positioning: it doesn't build agents, write their prompts, or choose their models; it models the *company* they work in. The tagline — "if OpenClaw is an employee, Paperclip is the company" — is also a precise statement of the architecture: Paperclip never reimplements the agent, it *employs* it, and the entire hard problem is the organizational coordination layer on top.

---

## Key Quotes

> "Hire AI employees, set goals, automate jobs and your business runs itself."

The thesis in a sentence, and it's the exact promise [[The Dead Economy Theory]] treats as the endgame of AI acceleration — productive capacity that "hums along without needing human participation." Paperclip is that idea shipped as an installable product, with the one deviation that a human *board* remains in the loop.

> "If it can receive a heartbeat, it's hired."

The bring-your-own-agent contract. Claude Code, OpenClaw, Codex, Cursor, shell commands, webhooks — Paperclip is unopinionated about runtimes. This is the same heartbeat primitive as [[Moltbook]]'s, but pointed at an internal org chart with budgets and an audit log rather than at an untrusted remote skill file. Same mechanism, opposite threat model.

> "Autonomy is a privilege you grant, not a default."

The governance stance, and the sharpest line on the page. Agents can't hire agents without board approval; the CEO can't execute an unreviewed strategy; config changes are revisioned and rollbackable. Paperclip keeps the human as the board — which is precisely the layer that pure "zero-human company" rhetoric usually deletes.

> "When they hit it, they stop. Automatically. No runaway costs. No surprise bills. Hard limits, enforced by the system."

Cost control as a first-class primitive, not an afterthought. Monthly per-agent budgets with atomic enforcement at checkout — the "runaway loop wastes hundreds of dollars of tokens" failure mode, made structurally impossible rather than admonished against.

> "We don't tell you how to build agents. We tell you how to run a company made of them."

The "What Paperclip is not" section is a masterclass in negative positioning: not a chatbot, not an agent framework, not a workflow builder, not a prompt manager, not a single-agent tool. "If you have one agent, you probably don't need Paperclip. If you have twenty — you definitely do."

## Key Themes

- **#concept** — The company as the unit of abstraction. Paperclip's object model is not tasks or pipelines but *organizations*: hierarchies, roles, reporting lines, goals, budgets.
- **#pattern** — Heartbeats as the scheduling primitive. Agents wake on a schedule or on notification (ticket assignment, @-mention) and delegate up and down the org chart automatically.
- **#pattern** — Governance with rollback. Approval gates, revisioned config, and the board as the top of the hierarchy — human-in-the-loop as a designed feature, not a safety patch.
- **#tool** — Bring-your-own-agent orchestration. Adapters to any runtime that can receive a heartbeat, sidestepping the framework wars entirely.
- **#concept** — Atomic budget enforcement. Task checkout and budget spend are one atomic operation, so no double-work and no runaway spend — the deterministic fix for a failure mode most orchestration tools merely warn about.

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

---

## Critical Analysis

**This is the org-science thesis turned into a product.** [[Multi-Agent AI Systems Are Organizations]] argues that multi-agent systems fail not from implementation bugs but from four universal organizing problems — task division, allocation, information, reward — and that they must be designed as organizations, not architectures. Paperclip *is* that prescription: its entire feature list (org charts, roles, reporting lines, goal alignment, budgets, governance, audit) is the four-problem framework rendered as a deployable runtime. Where the paper calls for a research program, Paperclip ships a Node.js process. The intellectual lineage is direct and unacknowledged on the landing page.

**"Zero-human company" is a deliberately provocative tagline, and the product quietly contradicts it.** The copy leads with "zero-human companies," but every governance feature keeps a human at the top: approve hires, review strategy, override budgets, pause or terminate any agent. What's actually being automated away is human *employees* — the board (and the budget-holder) remains. That's a meaningful difference from the fully-autonomous framing, and it's also the product's most defensible claim: it sells delegation with a kill switch rather than full autonomy.

**The bring-your-own-agent stance is smart positioning that also outsources the hard part.** By refusing to build agents, Paperclip avoids competing with Claude Code, OpenClaw, and the rest — it rides them. But it also means the safety and reliability of any given "employee" is someone else's problem ("your agents are your own and you secure them however you want to"). The governance layer Paperclip adds is real, but it governs *Paperclip* — who can hire, what strategy runs, what budget exists — not what an agent actually *does* inside its own runtime. That's a meaningful gap the landing page waves at but doesn't close.

**Where it slots into the orchestration landscape.** [[Agent Orchestration]] notes the field has mostly solved hierarchical role separation and is converging on kanban boards and atomic task claiming, but that governance, cost-control measurement, and multi-player orchestration remain largely unsolved. Paperclip is one of the few tools attacking exactly those gaps — atomic budget enforcement, revisioned governance, multi-company isolation. Whether it holds up against the [[Paca]]-style "agents as Scrum teammates" model or the [[Fleet Supervisor (sermakarevich)]]-style dumb-supervisor model is an open, empirical question; the landing page is a pitch, not evidence.

**The unresolved question is the same one the whole category faces.** [[OpenViktor]] was an "AI employee" that launched to #3 on Product Hunt, then was killed and rebuilt — the category's demand signal is real, but its execution risk is high. Paperclip's answer to that risk is to be boring infrastructure (a single Node process, embedded Postgres, MIT license) rather than a magical employee. That's the right instinct. Whether "run a company made of agents" survives contact with actual multi-week agent autonomy — drift, spec creep, the comprehension-debt problems [[Principal Drift]] names — is what a real deployment would test.

## Comparison notes

Among the meta-harnesses, Paperclip is the most *organizational* — and the least about the agent. [[QM (Multiplayer Agent Harness)]] also wraps Claude Code/Codex/Pi/OpenCode as company infrastructure, but standardizes the *tool surface* (a fixed set with `execute` as the escape hatch) and organizes around scopes (person/channel/team) with per-scope memory and keychains; Paperclip standardizes the *employment contract* — heartbeat + provider-neutral transcript + budget + org-chart position — and organizes around a literal reporting hierarchy. QM asks "what can this agent touch?"; Paperclip asks "who does this agent report to, and has it hit its budget?"

[[bb — The Agent Orchestrator as Normalizer]] is the closest engineering cousin, but one level down: bb normalizes *in-thread events* (provider events → shared `ThreadEvent[]`) at the protocol layer, with no governance or budgeting. Paperclip normalizes *run outcomes* (the `run_result` disposition) at the work-product layer, and spends most of its code on what bb leaves out — atomic checkout, recovery, budgets, approvals, audit logs.

[[Multi-Agent AI Systems Are Organizations]] argued that multi-agent systems face the four organizing problems of the Carnegie tradition by construction. Paperclip is the productization of that thesis: task division (issues + subtask trees), allocation (atomic checkout + assignee), information provision (goal ancestry flowing down every task), and reward distribution (per-agent budgets) are all first-class CRUD objects. It is also an existence proof for the paper's punchline — that these are solved by designing the *organization*, not the agent.

The [[Agent Orchestration]] hub's primitives all appear here as shipped features: atomic task claiming (the execution lock), worktree isolation (execution workspaces via git worktrees), ephemeral sessions with persistent state (session resume + task context across heartbeats), heartbeat loops, and kanban as the human interface. Paperclip's contribution is showing what happens when all of them are built into one coherent control plane with an org chart and a budget attached.

#tool #project #agents #orchestration #database #governance

---

*Sources: [[raw/paperclipai-net]], [[summary/paperclipai-net]], [[raw/paperclip]], [[summary/paperclip]]*
*Last updated: 2026-09-11*
