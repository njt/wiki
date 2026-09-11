# QM (Multiplayer Agent Harness)

QM is YC Software's open-source harness for running AI agents as *company* infrastructure, not personal assistants. Every person and every Slack room gets its own scoped memory, files, keychain, permissions, crons, and durable sandbox — and the same identity works across Slack and a web app. The thing that makes it interesting as engineering is the clean separation between a generic headless core and the harnesses that drive it: Pi, OpenCode, Codex, and Claude Code all plug into the same turn loop, so a deployment is never locked to one vendor.

---

## Architecture

**One composition root, interfaces everywhere.** `src/wiring.ts` (~1,600 lines) is the single wiring file the README advertises. Every store — sessions, runs, memory, files, keychain, rate limiter, budget, audit log — is an interface with an in-memory and a Postgres implementation, chosen by whether `DATABASE_URL` is set. The sandbox is its own interface (`src/sandbox/sandbox.ts`) with four backends (local, `sprites`, `smolmachines`, AWS microVM) routed per scope by `src/sandbox/sandbox-routing.ts`. This is ports-and-adapters done as a design centerpiece: the entire system can run in-memory for tests or Postgres-backed for production with one flag.

**The scope model is the spine.** `src/types.ts` defines `scopeId(kind, ref)` → `personal:<id>`, `channel:<ref>`, `team:<id>`, `group:<ref>`, `org:<id>`. Every resource hangs off a scope. `src/resolution/resolution-service.ts` turns (conversation, actor) into a `Resolution`: workspace layers (org read-only, scope read-write, team read-only), a system prompt assembled from org and scope "souls," an egress policy, a command policy, a security posture, and granted handles. The soul composition is deliberate: lower-scope instructions are wrapped with "may add to, but MUST NOT override, the organization policy above."

**Turns are queued work, not inline requests.** A turn becomes a "run" in a run store, processed by a pool of workers (`src/runs/worker.ts`, `config.workers`) with leader leases (`src/persistence/leader-lease.ts`), a drain controller, and a reaper. Crons ride `pg-boss`; monitors poll background processes. This is why the same core serves an interactive Slack mention and a 3am cron fire identically.

**The harness is an interface with a router.** `src/harness/harness.ts` defines `Harness` = profile + `turns` + `models` + `tools`. Five adapters implement it — pi (the default), opencode, codex, claude, mock — and `src/harness/harness-router.ts` picks one per turn from scoped config against an org-approved allowlist, resetting session state when a conversation switches harness mid-flight. The tool surface is defined once (`src/tools/primitives.ts`) and translated per harness; `src/harness/pi-tools.ts` (~2,900 lines) is the shared tool implementation reused by the Claude adapter.

**The agent loop lives in `src/core/orchestrator.ts`** (~3,000 lines): resolve scope → build prompt from soul + protocol markdown (`src/resolution/protocols/*.md`) → run the harness turn → apply memory strategy → deliver. Companion modules split compaction, prompt blocks, security screening, sandbox provisioning, and surface tools out of it.

---

## Key techniques

**A small, fixed tool surface with `execute` as the escape hatch.** The model gets roughly `execute`, `read`, `write`, `publish`, `memory`, `history`, `background`, plus MCP and cron/surface control. `execute` runs commands in the scope's sandbox — its durable "computer" where installed tools stay installed. This inverts the usual agent-tool sprawl: instead of exposing dozens of bespoke tools, QM exposes a shell and lets skills/installed tools do the rest, which is what makes swapping harnesses tractable.

**Tool-result ledger as a cache.** `src/tools/primitives.ts` wraps most tool calls in `once()`, which records a result keyed by (run, attempt, callIndex) so a retried attempt replays the identical result instead of re-running a command. It's a correctness guard for at-least-once delivery, not a perf trick.

**Provenance-disciplined memory extraction.** The default `per-turn` strategy (`src/memory/strategies/per-turn.ts`) buffers turns and asks a model to extract durable facts, but its prompt is explicit that a preference is a fact *only if the user's own message states it* — an assistant saying "per X's preference" is not evidence. Autonomous turns get a stricter addendum: operational facts only, never preferences. `scratch-promote` (`src/memory/strategies/scratch-promote.ts`) writes captures to dated log files, then promotes durable ones into a `MEMORY.md` notebook via an LLM rewrite protected by compare-and-set revision checks. Consolidation (`src/memory/strategies/consolidation.ts`) reduces the notebook with `UPDATE n:` / `DELETE n:` / `ADD:` actions rather than regenerating it.

**Security as a rank-ordered posture with a classifier.** `src/security/security-posture.ts` maps three postures to two axes (inbound screening on/off, tool approvals none/all) and composes them by rank so a scope can only tighten the org floor. In `auto`, a prompt-injection classifier (`src/security/security-screener.ts`, or an external screening proxy) judges provenance-labelled external data and returns `{"decision":"auto"|"strict"}`. The command policy (`src/policy/command-policy.ts`) is a denylist/allowlist evaluated before every `execute`, with hard denials that survive even `dangerous`.

**Capability tokens for egress.** The sandbox talks to the outside through an egress proxy authenticated by short-lived JWTs (`src/auth/capability-token.ts`) that encode the scope and allowed hosts, minted per turn.

---

## Design decisions

**Optimized for tenancy and auditability over raw autonomy.** The cost of per-scope memory/keychain/sandbox isolation is a lot of machinery, but it's what makes an org trust the agent with real credentials. The README's "acts as the person it's working for, with their credentials and permissions, and everything it does is audited" is the framing — local-coding-agent trust model, applied company-wide.

**Harness interchangeability over harness features.** By defining one fixed tool surface and translating it, QM trades away each harness's native tool richness for the ability to swap Pi→Codex→Claude Code per turn. The harness-router even resets session state on a mid-session switch — a clean admission that harnesses are stateful and not freely mixable.

**Deployment directory vs. private fork.** Two ways to customize, both engineered to keep core byte-identical to upstream so merges stay small. The README is unusually precise about why *not* to use GitHub's fork button (a GitHub fork inherits visibility and shares an object network, so private forks leak commits by SHA).

**Sacrificed: a lot of conventional tool surface.** The `execute`-centric design means the model needs decent shell fluency, and the harness adapters must each re-implement the tool bridge — visible in the four 800–2,900-line `*-harness.ts` files.

---

## Comparison notes

This is the open-source, self-hosted answer to [[Introducing Claude Tag]]: where Anthropic ships a single-vendor, channel-scoped team agent, QM ships the same "agent in Slack" shape as MIT-licensed infrastructure with a full scope hierarchy (org/team/channel/group/personal) and any model behind it. See [[Introducing Omnigent]] and [[bb — The Agent Orchestrator as Normalizer]] for other takes on wrapping multiple harnesses — QM inverts Omnigent by standardizing the *tool surface the harness sees* rather than the user-facing I/O. [[Munder Difflin — Clones of You, Not a Shared Bot]] pushes the same multi-harness wrapping to the opposite architectural pole: instead of centralizing per-scope memory, keychain, and sandbox on a server, it runs each clone as a peer node on its owner's laptop and lets clones talk over end-to-end-encrypted messaging — the distributed, no-server trust model where QM's answer is an operated server.

[[The Agent Access Model]] is the most direct conceptual neighbor: QM's scope + ACL-grant + keychain + command-policy stack is a concrete, running attempt at the "multiplayer access control" that Cloudflare Research said "we are not comfortable saying can be built end to end today." QM's posture-composition (org floor, scopes can only tighten) echoes AAM's Trust Ratchet spirit at coarser granularity. Also relevant: [[Security and Sandboxing]] (the four sandbox backends), [[Agent Memory and Context]] (the notebook/scratch-promote tiering), and [[Cloudflare OS]] (another org-wide agent platform with per-user sandboxes).

[[Paperclip]] is the same "agents as company infrastructure" bet with the emphasis inverted: QM organizes around scopes and gives each agent a memory, keychain, and sandbox; Paperclip organizes around an org chart and gives each agent a role, a boss, and a monthly budget. QM optimizes for tenancy, Paperclip for governance.

#tool #project #agents #memory #security #slack

---
*Sources: [[raw/qm]], [[summary/qm]]*
*Last updated: 2026-08-14*
