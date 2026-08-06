# Building an Advanced Agentic Harness

A Data for Science tutorial that upgrades a basic agent loop into a production-shaped harness through seven composable primitives — typed tools, plan DAG, tiered memory, verification hierarchy, Planner/Worker/Critic roles, multi-dimensional budgeting, and a tracer — each motivated by a specific failure mode of naive agents. The thesis: composability, not framework magic, is what separates a proof-of-concept from an extensible harness. #concept #pattern #tool

---

## Key Quotes

> "Composability is the difference between a PoC and an extensible harness, and is why the orchestrator stayed thin."

The article's thesis compressed. Unlike LangChain or similar frameworks that absorb complexity into opaque abstractions, every primitive here is independent and swappable. Add a tool to the registry and the planner automatically sees its schema. Tighten `verify_report()` and every future run is held to the new bar. Swap the entire LLM backend and the harness reruns unchanged. This is the engineering philosophy that [[Harness Engineering (OpenAI)]] operationalized at scale — each component earns its keep independently.

> "A bad plan should fail fast, at the validation layer, and not deep inside a database query."

The argument for typed tools with Pydantic validation. The LLM emits a plan; the harness validates it structurally before spending a single token on execution. Hallucinated node IDs, circular dependencies, invalid argument shapes — all caught at zero cost. This is the same principle as [[Guardrails and Feedback Loops]]'s core claim that linters beat prompts, applied to the planning layer rather than the output layer.

> "Context should be actively assembled, not passively accumulated."

The tiered memory thesis. Working memory is always in context; episodic memories (past similar tasks) are retrieved top-k by embedding similarity; semantic memories (background facts) fill remaining budget. When budget runs out, truncation is explicit — the system says `(...truncated at budget...)` rather than silently dropping content. Episodic beats semantic in retrieval priority because past mistakes on similar tasks are more actionable than generic facts. This is a concrete, code-backed implementation of the patterns surveyed in [[Agent Memory and Context]], with the retrieval-budget mechanism adding a constraint most memory systems lack.

> "The rule is to always run the cheap tier first and only escalate survivors."

The verification hierarchy in one sentence. Deterministic structural checks (e.g., "did every requested city appear in the report?") cost zero tokens and catch obvious failures. Only outputs that survive the cheap tier reach the expensive LLM judge. This is the pattern behind most production eval pipelines, and it's the same two-tier gate that [[Cloudflare Security Audit Skill]] uses (deterministic grep → LLM confirmation) and that [[Metis — ARM AI Security Code Review]] formalizes (tree-sitter reachability → LLM vulnerability confirmation).

> "Pressure is the maximum utilization across all dimensions. You are limited by whichever resource runs out first, exactly like real billing."

The multi-dimensional budget insight. A single `max_steps` counter hides the real constraints: you can have steps left but no tokens left, be under token budget but rate-limited on tool calls, or have a hung network call burn wall-clock time. `BudgetMulti` tracks tokens, tool calls, wall time, and estimated dollars simultaneously, with a single `pressure()` scalar that drives graceful degradation — skip the LLM judge above 0.9, halt at 1.0. Production agents use the same signal to switch to cheaper models, reduce retrieval depth, or ask for confirmation.

> "Never take the LLM at its word, not even about node names."

A hard-won operational lesson. The mock planner always named the capstone node `aggregate`. A real model sometimes mirrors the tool name (`aggregate_report`) or invents its own id. The fix: resolve the capstone by tool rather than by hardcoded id, and add explicit instructions to the planner prompt. This is the kind of brittleness that separates demos from production — and why [[Components of a Coding Agent]]'s taxonomy matters: the harness is doing real work that naive implementations miss.

---

## The Seven Primitives

### 1. A Pluggable Brain (`LLMProvider`)

Before building anything, abstract the LLM call behind an interface. A `MockProvider` returns deterministic, role-aware responses (canonical plan when asked to plan, templated summary when asked to summarize, rule-based pass/fail when asked to judge). This separates "is my orchestration wrong?" from "is the model planning badly?" — the same separation that [[Sherlock Agent Eval]] formalizes as a benchmark methodology.

### 2. Typed Tools

Each tool's arguments are a Pydantic model that drives everything: runtime validation, JSON Schema for the LLM, documentation from `Field(description=...)`, and a `cost_hint` for budget accounting. Failing before execution avoids expensive calls with bad arguments. This converges with what Anthropic tool use, OpenAI function calling, and LangChain tools all do, but keeps it as a single `@dataclass` rather than a framework.

### 3. The Plan is a Graph

Instead of asking the LLM for one action at a time, ask the Planner for the whole DAG up front. Validate it structurally (no circular deps, no references to nonexistent nodes) before executing. A level-synchronous executor with `asyncio.Semaphore` runs all ready nodes concurrently. For the city comparison task: nine independent fetches run in parallel, then one aggregation. Sequentially, wall time is roughly the sum of latencies; in parallel, it's roughly the max of them plus the aggregation.

### 4. Tiered Memory

Working memory (always in context) → episodic (past similar tasks) → semantic (background facts). Retrieved top-k by embedding similarity, assembled under a hard character budget, with episodic prioritized over semantic. The store supports both Jaccard similarity (free, fails on paraphrase) and sentence embeddings via `all-MiniLM-L6-v2` (handles paraphrase, costs compute). Real benchmarks deferred to a future post — the article is honest about what it measures and what it doesn't.

### 5. Verification Hierarchy

Cheap deterministic checks first (structural: missing cities, format violations), expensive LLM judge only on survivors. The Worker produces; the Critic evaluates. The generator never grades its own homework. This is the same separation that [[Accordant]] formalizes ("the spec IS the oracle") and that [[OpenCodeReview]] implements at scale with per-file concurrent subagents.

### 6. Planner / Worker / Critic

Three narrow agents, each with a short system prompt and a single contract. The Planner gets the goal + tool schemas → DAG JSON. The Worker receives the DAG and executes. The Critic receives the goal + finished report → verdict. All three go through the same `LLMProvider.complete(..., role=...)` interface. This is the same pattern that [[Agent Orchestration]] identifies as recurring independently across Cursor, Hightouch, and Maestro — but implemented here as ~50 lines of Python rather than a multi-service architecture.

### 7. Multi-Dimensional Budgeting

`BudgetMulti` tracks tokens, tool calls, wall time, and estimated dollars. A single `pressure()` scalar drives graceful degradation. `classify_error()` maps errors into four classes (transient, tool misuse, missing info, policy violation) with recovery policies that follow from the class rather than blind retries. Transient errors get exponential backoff with jitter; missing-info errors trigger informed re-planning; policy violations halt immediately.

### 8. The Tracer (Flight Recorder)

Flat, boring JSON events capturing identity (step ID, parent ID), semantics (role, action), economics (latency, tokens, cost, budget pressure), and verdicts. The schema is simple enough to dump to a file, ship to OpenTelemetry, or plot in matplotlib. Per-step latency by role shows LLM calls dominate wall time; tokens by role shows whether planning or execution consumes the budget; budget pressure over time shows if degradation thresholds were crossed.

---

## Critical Analysis

**The article's greatest strength is its pedagogical structure.** Each primitive is introduced with the specific failure mode it addresses ("LLMs invent invalid tool arguments, so we add typed tools"), built incrementally, and shown in composable isolation. The running example (city comparison) is deliberately trivial so the architecture — not the domain — stays in focus. The mock provider means every experiment is reproducible without API keys. This is how technical tutorials should be written.

**The "framework-free" stance is both a strength and a limitation.** By showing every primitive as explicit Python rather than LangChain/LlamaIndex abstractions, the article teaches harness engineering from first principles. But it also means readers must reimplement concepts that mature frameworks have already solved (retry with backoff, structured logging, model routing). The article's own caveat acknowledges this: "memory here is in-process, while production systems persist embeddings to Chroma, Weaviate, or pgvector." The question is whether the pedagogical value of showing the internals outweighs the practical gap between the tutorial implementation and what production requires. For learning, yes. For shipping, you'd want to wrap these patterns in battle-tested infrastructure.

**The Planner/Worker/Critic pattern here is notably thin compared to production systems.** The Planner emits a single static DAG — there's no dynamic replanning mid-execution, no sub-goal decomposition, no hierarchical planning. Cursor's [[Scaling Long-Running Agents]] needed recursive task decomposition; Hightouch's [[How Hightouch Built Their Long-Running Agent Harness]] needed dynamic context compression. The article's pattern is the right starting point, but the gap between this and production is the same gap between a single-level DAG and a recursive task tree.

**The verification hierarchy is correct but incomplete.** The two-tier gate (deterministic → LLM judge) catches structural failures and subjective quality issues. It doesn't address correctness failures that pass structural checks — a report that includes all cities but fabricates population numbers passes the deterministic tier and would need a different kind of judge. This is where [[Ways of Checking]]'s ten verification failure modes become relevant: "checking again re-runs the instrument; checking differently tests it."

**The omission of evals is honest but significant.** The article explicitly defers eval infrastructure to a future post, and the final caveat is unusually candid: "A single successful demo proves the harness can work; nothing in this post proves it actually works in most cases." This is the right intellectual honesty, but it also means the article teaches harness construction without teaching harness validation — and as [[Goodhart's Law and AI Benchmarks]] argues, unvalidated harnesses are just elaborate anecdotes.

**The budget pressure mechanism is elegant and under-explored.** The `pressure()` scalar as max-utilization-across-dimensions is a clean abstraction that most production systems approximate with ad hoc checks. The graceful degradation ladder (full pipeline below 0.7 → skip critic above 0.9 → halt at 1.0) is a template that could generalize to many domains. The article doesn't explore what happens when different dimensions have different degradation curves — token budgets deplete smoothly, rate limits are step functions — but the framework is extensible.

**The article pairs well with several wiki pages.** [[Components of a Coding Agent]] provides the taxonomy this article implements — Raschka defines what a harness is; this article shows how to build one. [[Harness Engineering (OpenAI)]] is the production field report; this article is the engineering tutorial that makes the field report's abstractions concrete. [[Loop Engineering]] describes the practice of designing systems that prompt agents; this article provides the primitives those systems are built from. And [[Harness Engineering for Self-Improvement]] surveys the research trajectory where these handcrafted primitives become optimization targets — the article's composable architecture is exactly the kind of harness code that Darwin Gödel Machine and AHE would evolve.

**The re-planning loop is the most underrated contribution.** When execution fails because of missing information (a hallucinated city name), the harness doesn't blindly retry — it sends the failure context back to the Planner for informed re-planning. This is a concrete implementation of the error-classification pattern that [[Harness Engineering is not Enough]] argues is essential: not all failures are retry-eligible, and retrying the wrong class of error is actively harmful. The `classify_error()` function mapping errors to transient/tool-misuse/missing-info/policy-violation is the kind of small, high-leverage primitive that separates robust systems from fragile ones.

---

*Sources: [[raw/building-an-advanced-agentic-harness]], [[summary/building-an-advanced-agentic-harness]]*
*Last updated: 2026-08-06*
