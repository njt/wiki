# Loomkin

A multi-agent platform built on Elixir/Erlang OTP where AI agents form teams as fluidly as humans. Each agent is a GenServer under a DynamicSupervisor -- spawn in under 500ms, communicate via PubSub in microseconds, support 100+ concurrent agents per node. The BEAM VM's concurrency model, designed for telecom switches, turns out to be ideal for agent orchestration.

---

## Key Themes

#orchestration #agent-architecture #agentic-coding

What distinguishes Loomkin from single-agent assistants:

**Decision graph** -- a persistent DAG of goals, decisions, and outcomes (7 node types, typed edges, confidence scores). Not chat history -- structured reasoning memory that survives across sessions. Cascade uncertainty propagation warns downstream nodes when confidence drops.

**Context mesh** -- instead of summarizing context away as the window fills, agents offload to Keeper processes with staleness tracking and failure memory. 228K+ tokens preserved vs. 128K with zero loss. Knowledge that becomes stale auto-archives. Failure memory keepers capture lessons from errors.

**Self-healing teams** -- error classification, agent suspension, ephemeral diagnostician + fixer agents that repair failures autonomously. Per-role healing policies. OTP supervision trees make this natural.

**Conversation agents** -- any agent can spawn a freeform multi-agent conversation: brainstorm, design review, red team exercise. A Weaver agent auto-summarizes the outcome. Deliberation as a service.

**Verification loops** -- autonomous write-test-diagnose-fix cycles. Upstream verifiers auto-spawn on task completion to validate output before dependents proceed.

Five built-in roles (lead, researcher, coder, reviewer, tester), 58 built-in tools, 16 LLM providers, 665+ models. LiveView web UI with 39 components and zero handwritten JavaScript. 328 source files, ~77K LOC, 2,700+ tests.

The BEAM/OTP choice vindicates the argument in [[Systems Ideas That Sound Good]] -- Sinofsky warned against DIY async, and Loomkin's answer is: don't do it yourself, use a platform built for it. This is also the approach [[Software Craft]] references via BEAM process-based concurrency.

## Critical Analysis

Loomkin is the most ambitious multi-agent system in this batch. The architecture is genuinely differentiated -- OTP supervision, process-level isolation, microsecond messaging, hot code reload. These aren't features bolted onto a Python script; they're intrinsic to the platform.

The concern: 77K LOC and 328 source files is a large system. The Elixir ecosystem is smaller than Python/TypeScript, which limits the contributor pool. And the comparison table in the README claims advantages over "traditional AI assistants" that are partly architectural and partly marketing -- not every comparison is apples-to-apples.

Still, the core bet -- that BEAM concurrency is the right substrate for multi-agent orchestration -- is one of the most interesting technical bets in the space. The decision graph and context mesh alone are worth studying.

---
*Sources: [[summary/loomkin]]*
*Last updated: 2026-05-14*