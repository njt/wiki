# Thrifty (Tiered Delegation for Claude Code)

2389 Research's Claude Code plugin that implements tiered-delegation execution: Sonnet plans work into sprints with a contract, Haiku builds and self-verifies against gates, and Sonnet only steps in when Haiku fails. Benchmarked at ~64% cheaper than Opus at equal quality on multi-unit builds — the cost savings come from replacing expensive model token consumption with cheaper model token consumption for the bulk of the work, plus a verification architecture where passing gates cost zero model tokens.

---

## Key Quotes

> "Offload each sprint of work to the weakest model that can do it correctly."

This is the guiding principle, and it's a sharper formulation than it first appears. "Weakest model that can do it correctly" forces you to define "correctly" before you start — that's where the gates come from. The architecture doesn't work without a contract that pins what success looks like at sprint granularity. This is specification-as-scaffolding, not specification-as-documentation. The spec *is* the delegation interface.

> "~64% cheaper than Opus building the same spec, at equal gate quality."

The qualification "at equal gate quality" is doing real epistemological work. They're not claiming the output is as good in some holistic sense — they're claiming the same tests pass. This is honest about what's being measured: gate-passing, not judgment. Code that passes the same tests isn't necessarily code of the same quality. But for many production tasks, passing the same tests is sufficient.

> "The common path costs no checker tokens."

The adaptive verification design — runnable gates (tests, builds, CLI exit codes) just execute without model involvement; Sonnet only reads and fixes on failure — is the smartest single design choice in the system. It means the verification tax is proportional to failure rate, not task count. For codebases with good test coverage, most sprints pass their gates on the first try, and the expensive Sonnet checker never spins up.

> "Every subagent spawn carries ~40k of Claude Code harness context, and each report re-enters the orchestrator's context, compounding cost with unit count."

This is the dispatch-vs-subagent tension in a single sentence. The subagent path is architecturally cleaner — proper agent delegation with context isolation — but it carries a ~40k-token overhead *per sprint* just for the harness. At that rate, the subagent substrate's overhead can exceed the cost of just running Sonnet directly. The dispatch flow (`claude -p` calls hard-pinned to Haiku) avoids this by using bare CLI calls and a tiny manifest. The cost is elegance; the gain is actual savings.

> "The subagent path cannot force the executor model in code."

An honest admission that undercuts the architecture's promise. If your runtime silently falls back from Haiku to Sonnet, your cost ledger is fiction. The dispatch path solves this by using `claude -p --model haiku` — explicit model pinning that can't be overridden. This is a microcosm of a larger problem in agent architectures: cost estimates that depend on model selection are only as reliable as the enforcement mechanism for that selection.

---

## Key Themes

#pattern #tool #cost-optimization #multi-model #delegation #claude-code-plugin

### The Inversion of the Advisor Pattern

Thrifty and Anthropic's [[The Advisor Strategy]] attack the same problem (cost reduction through model tiering) from opposite directions. The Advisor Strategy has a cheap executor (Haiku/Sonnet) driving with an expensive advisor (Opus) on call. Thrifty has an expensive planner (Sonnet) decompose the work, then a cheap executor (Haiku) does the bulk. Advisor is bottom-up escalation; thrifty is top-down delegation. They're complementary: Advisor is better for tasks where you can't decompose in advance; thrifty is better for tasks where you can write a spec upfront.

### Gates as the Trust Contract

The insight that gates should be independently re-run rather than self-reported by the executing agent is fundamental. Self-reported verification — "I ran the tests and they passed" — is the weakest link in every agent workflow. Thrifty sidesteps this entirely: the orchestrator runs the gate itself, and only involves a Sonnet checker when something fails. This is the same principle as [[Guardrails and Feedback Loops]]: deterministic enforcement beats probabilistic pleading.

### The Subagent Overhead Problem

The dispatch-vs-subagent tension in thrifty reveals a real architectural issue in Claude Code plugins. Subagents are the natural abstraction for delegation — each sprint as an independent agent with its own context — but the harness overhead (~40k tokens per spawn) makes them uneconomical for fine-grained work. The dispatch flow's use of bare `claude -p` calls is functionally a workaround for this overhead. It calls into question whether subagents are the right primitive for cost-sensitive delegation, or whether we need a lighter-weight agent spawn that doesn't carry the full harness. See also [[Components of a Coding Agent]] and [[Maybe Coding Agents Don't Need a Bigger Memory]].

### Three Decomposition Modes

The Partition/Relay/Layered taxonomy is useful beyond this tool. Partition (parallel, separate files) maps to embarrassingly parallel coding tasks. Relay (sequential, shared artifact) maps to document generation. Layered (sequential, different lenses on the whole) maps to editing and refinement passes. Most agent orchestration discussions default to Partition; thrifty's explicit support for Relay and Layered modes acknowledges that not all work decomposes into independent units.

---

## Critical Analysis

Thrifty is the most carefully benchmarked attempt I've seen to make multi-model orchestration actually save money rather than just rearrange it. The 64% claim is backed by published eval results, and the honesty about when the savings evaporate (trivial tasks, subagent-mode overhead, model-enforcement gaps) is refreshing.

But the tool also reveals how fragile cost-optimization architectures are. The savings depend on: (1) your runtime respecting model pinning, (2) your task being decomposable into independent sprints, (3) your gates being runnable rather than assertional, (4) your Haiku actually being capable enough to pass those gates. Remove any one of these and the cost advantage collapses — sometimes below zero.

The deeper question thrifty raises is whether this architecture should exist *inside* the model API rather than as a plugin. Anthropic's Advisor tool (`advisor_20260301`) moves model-tiering logic server-side, where cost optimization happens transparently. Thrifty moves it client-side, where you can tune it but also break it. The trade is control vs. reliability — and for a plugin that's all about cost savings, reliability of the savings claim matters more than tunability of the mechanism.

The comparison to [[StrongDM Factory Techniques]] is instructive. StrongDM's dark factory doesn't try to optimize which model does what — it optimizes the harness around a single model. Thrifty optimizes model selection. Both work. The question is which approach ages better as model pricing converges.

**The bottom line:** If you're building multi-file features with good test coverage, thrifty's dispatch flow will likely save you money. If you're doing exploratory work, single-file changes, or anything where a spec would cost more to write than the code, skip it — the planning overhead will eat your savings. The tool knows this and says so. That self-awareness is its best feature.

---

*Source: https://skills.2389.ai/plugins/thrifty/, fetched 2026-07-03*
*See also: [[raw/thrifty]]*
