# The Advisor Strategy

Anthropic's formalization of a cost-intelligence architecture where a smaller executor model (Sonnet or Haiku) drives agent tasks end-to-end, escalating to a powerful advisor model (Opus) only on critical decisions. The advisor reads shared context, returns a plan or correction, and the executor continues — no tool calling or user-facing output from the advisor. It inverts the standard orchestrator-worker pattern: the cheap model steers, the expensive model rides shotgun. Announced April 2026 as a native `advisor_20260301` tool on the Claude Platform, requiring a one-line API change.

---

## Key Quotes

> "Frontier-level reasoning becomes a resource the executor can call on demand rather than a fixed cost paid on every turn."

This is the economic insight. Intelligence becomes something you meter, not something you marry. The advisor pattern treats reasoning as a utility — dial it up when the task gets hard, dial it down when it's routine. This is genuinely different from the prevailing "pick a model and commit" approach.

> "This inverts a common sub-agent pattern, where a larger orchestrator model decomposes work and delegates to smaller worker models."

The inversion is the architecture. Everyone builds top-down orchestration (big brain → small hands). Anthropic is proposing bottom-up escalation (small brain → big brain on demand). It's simpler, has fewer failure modes, and doesn't require decomposition logic. The junior engineer metaphor — does most of the work, occasionally taps a senior on the shoulder — is the right intuition.

> "Sonnet with Opus as an advisor showed a 2.7 percentage point increase on SWE-bench Multilingual over Sonnet alone, while reducing cost per agentic task by 11.9%."

The counterintuitive result: *adding* an expensive model *reduces* cost. The mechanism: Opus produces only ~400–700 tokens per consultation (short plans, not full trajectories), and its guidance prevents the executor from wasting tokens on dead-end iterations. Intelligence pays for itself by preventing stupidity.

> "Haiku with an Opus advisor scored 41.2%, more than double its solo score of 19.7%. Haiku with an Opus advisor trails Sonnet solo by 29% in score but costs 85% less per task."

Haiku + Opus advisor is the most interesting combination. For tasks where you can tolerate a quality gap relative to Sonnet, you get a dramatically better cost profile. This isn't just about making Sonnet slightly better — it's about making Haiku viable for production workloads that previously required Sonnet.

> "It makes better architectural decisions on complex tasks while adding no overhead on simple ones."

The zero-overhead-on-simple claim is critical. If the advisor adds latency or cost to every task — even trivial ones — it's a non-starter for production. The `max_uses` guardrail (recommended: 3) and the executor's discretion on when to invoke combine to create a ceiling on advisor cost per task.

---

## Key Themes

- **#pattern** Advisor-executor architecture — A new entry in the agent topology catalog. Unlike planner-worker (top-down decomposition) or swarm (peer-to-peer), the advisor pattern is bottom-up escalation: a competent executor that knows when it's out of its depth. The executor retains full agency; the advisor is purely consultative.

- **#concept** Intelligence composability — Model capability stops being a property of a single model's weights and becomes a property of a model *pair*. The effective intelligence of a Sonnet executor is (Sonnet + Opus-on-demand). This opens a design space where you don't choose a model — you choose a spectrum, tuning the advisor's `max_uses` and invocation triggers.

- **#tool** `advisor_20260301` — The first server-side tool type in the Messages API that changes which *model* processes the turn. Declared like any other tool, the executor decides when to invoke it, and Anthropic's servers handle the context routing and model handoff. No client-side orchestration required.

- **#concept** The intelligence-to-cost ratio becomes continuous — Before: you pick Haiku (cheap, weak), Sonnet (mid), or Opus (expensive, strong). After: you pick an executor, attach an advisor, and set `max_uses`. The cost-capability tradeoff becomes a slider, not a dropdown. This is the commercial flywheel — it incentivizes using *multiple* Anthropic models rather than picking one.

- **#pattern** Server-side context sharing — The advisor reads the same conversation context as the executor. No separate context management, no window duplication, no synchronization logic. This is an underappreciated design decision: the simplicity comes from the server owning the handoff, not the client.

---

## Critical Analysis

**This should have been obvious sooner.** The advisor pattern is so clean you wonder why nobody productized it before. The answer is probably API architecture: most model providers expose stateless endpoints. Building the advisor pattern *properly* — shared context, server-side routing, native tool type — requires the provider to own the agent loop, not just serve tokens. Anthropic can do this because they've been building toward agents-as-a-platform (MCP, tool use, extended thinking). It's a move that's impossible for a pure model-as-a-service provider to copy without rebuilding their API surface.

**The `max_uses` guardrail is doing more work than it looks like.** Three advisor consultations per task means the cost is bounded regardless of task length. This isn't just about cost control — it's about making the pattern *predictable*. You can budget for it. You can put it in a production SLA. Without `max_uses`, the advisor strategy is a research demo; with it, it's a production feature. Simple constraints unlock real-world adoption.

**The benchmark results are encouraging but narrow.** SWE-bench Multilingual, BrowseComp, Terminal-Bench 2.0 — these are coding and search benchmarks. We don't have data on how the advisor pattern performs on creative writing, analysis, or multi-step enterprise workflows. The pattern *should* generalize (the architecture doesn't care what domain the task is in), but we've seen enough benchmark-to-reality gaps to be skeptical. The design partner quotes suggest it works in practice, but those are selected testimonials.

**The hidden cost is the model's judgment about when to escalate.** The entire pattern depends on the executor model correctly identifying when it needs help. A Haiku executor that *doesn't know what it doesn't know* will fail silently without ever invoking the advisor. A Sonnet executor that's *too eager* will burn advisor calls on trivial decisions. The calibration of escalation threshold is the invisible engineering problem — and it's entirely in the model's hands, not the developer's. You can tune the system prompt, but you can't directly control the invocation decision.

**This is a commercial move disguised as an architecture pattern.** The advisor strategy isn't just good engineering — it's good business. It creates a reason to use Opus (advisor tier), Sonnet (executor tier), and Haiku (budget executor tier) together. Before this, you had to choose one model and optimize around it. Now the optimal setup is *multiple Anthropic models in the same request*. The technical elegance is real, but so is the lock-in mechanism.

**The comparison to Step 3.7 Flash's Advisor Mode is instructive.** StepFun shipped a similar pattern (small executor escalates to frontier) as a product feature for their own model family almost simultaneously. The fact that two independent labs landed on the same architecture within weeks of each other suggests this isn't just marketing — it's a genuine architectural convergence. The difference: Anthropic owns the API layer, so their advisor is a native tool type rather than a client-side pattern. That's a moat.

**What's missing:** No data on latency impact of advisor consultations. No ablation studies (does the improvement come from Opus's intelligence or just from having a second opinion?). No guidance on how to tune the system prompt for different escalation thresholds. No cost data at production scale (the 11.9% reduction is per-task, but what about tasks where the advisor fires all three `max_uses`?). The announcement is an announcement — the real engineering comes when people stress-test this in production.

---

## Related Pages

- [[Step 3.7 Flash]] — StepFun's Advisor Mode: the same architecture, shipping independently. The comparison reveals what's genuinely convergent vs. what's Anthropic-specific
- [[Agent Orchestration]] — The advisor pattern as a new entry in the orchestration taxonomy: bottom-up escalation vs. top-down decomposition
- [[Smart Models Dumb Pipes]] — Models own decisions, pipes own execution. The advisor pattern is a direct application: the executor owns the loop, Opus owns the hard calls
- [[Honey I Shrunk the Coding Agent]] — The scaffold matters more than the model. The advisor pattern is scaffold innovation at the API layer
- [[Components of a Coding Agent]] — The harness > model thesis. The advisor tool changes the harness's relationship to model capability
- [[Computer Use is 45x More Expensive Than Structured APIs]] — The cost argument that makes the advisor strategy economically necessary
- [[Scaling Long-Running Agents]] — Context management in long agent runs; advisor consultations are punctate interventions in long trajectories
- [[Building Agents for Production Systems with MCP]] — Anthropic's production agent guidance; the advisor tool is the next piece of that platform
- [[Agent-Native Architectures (Every)]] — Design principles for agent systems; the advisor pattern maps to parity (same context) and composability (stackable models)
- [[Elements of Agentic Systems Design]] — Ten-element taxonomy; the advisor pattern touches Agency, Reasoning, and Coordination
- [[Compound Engineering]] — Cost-intelligence tradeoffs as a first-class engineering concern
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management as the real engineering challenge; the advisor avoids context duplication
- [[All Your Agents Are Going Async]] — Agent transport patterns; the advisor runs synchronously within a single request
- [[Building Production-Ready Voice Agents]] — 50% of effort goes to admin, not the agent; the advisor's `max_uses` simplifies the admin burden
- [[Agent Orchestration for the Timid]] — The advisor pattern as the timid orchestrator's dream: no decomposition, no worker pool, just occasional escalation

---

*Sources: [[raw/the-advisor-strategy]]*
*Last updated: 2026-06-09*
