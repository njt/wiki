# Reducing Token Spend with Deterministic Workflows

Adam Jacob's field report on replacing an LLM-coordinated code review pipeline with a deterministic Swamp workflow: 8× fewer tokens, 2× faster execution, and better failure behavior. The practical methodology and the deeper argument it rests on — use the agent to build the program that minimizes the need for the agent.

---

## One-Paragraph Precis

When David Cramer reported Sentry was burning $10,000/week on AI tokens, Adam Jacob rebuilt his Garfield code review extension from the ground up. The original design used a coordinator LLM dispatching 23 sub-agents in a non-deterministic loop, consuming ~4.5M tokens over 12 minutes. Jacob's rewrite — a deterministic Swamp workflow with exactly 3 agents — cut that to ~500K tokens and 6.5 minutes. The methodology (understand → translate to deterministic code → black box test → refactor) is replicable, and the deeper thesis — "you use the Agent to build the program that minimizes the need for the Agent itself" — is the most important engineering discipline in agentic development right now.

---

## Key Quotes

> "You use the Agent to build the program that minimizes the need for the Agent itself."

The article's thesis compressed into one sentence. This is the inversion that separates maturity from vibes: stop treating the LLM as the runtime and start treating it as the compiler that produces a deterministic program. The program runs cheap; the compiler was expensive once.

> "The coordinator was doing a lot of work that didn't need to be done by an LLM."

The diagnostic that generalizes beyond Garfield. If your agent workflow has a "coordinator" agent deciding what to do next, interrogate whether that decision actually requires intelligence or is just control flow wearing a chat interface. Most coordinator loops are state machines in disguise — and state machines are cheaper to run in code than in tokens.

> The original skill "failed *open*" while the workflow "failed *closed*."

Buried in the benchmark table but arguably the most important result. The original skill declared success on a codebase it didn't fully fix — a silent failure. The deterministic workflow reported unresolved findings instead of pretending it was done. Jacob calls this guardrailing "a desirable fail-secure pattern." Correct, and underappreciated: deterministic code fails better than LLM code because it has fewer places to silently swallow errors.

> "I still had to refactor. Even when you use AI."

Blunt and necessary. The agent produced a working translation, but it wasn't clean. Jacob emphasizes that Swamp workflows are refactorable *because* they're "normal" deterministic code — you can improve them the way you improve any program, rather than prompting your way through a labyrinth of conversation history.

> "Lex Aedificandi, Lex Credendi" — as we build, so we believe.

The closing aphorism. Jacob's point: the architecture you choose isn't neutral; it shapes what you think is possible. If you build with LLM-in-the-hot-path, you believe intelligence must be everywhere. If you build with deterministic orchestration and targeted agent delegation, you believe intelligence should be surgical. Build the second way and your beliefs align with your costs.

---

## Key Themes

#pattern #concept #tool #cost-optimization

- **LLM-as-compiler, not LLM-as-runtime** — The central architectural inversion. Use the agent to *produce* a program (workflow, state machine, typed pipeline), then run that program deterministically. The LLM is expensive but invoked once; the program is cheap and invoked forever. This is the same insight as [[Specifications as the Product]] applied to the agent's own execution model.

- **Coordinator agents are an anti-pattern** — When an LLM decides "what should I do next?" in a loop, you're paying token costs for control flow that could be a DAG. Jacob's 23-agent coordinator → 3-agent deterministic pipeline is the worked example. The [[Agent Orchestration]] hub documents the planner/worker/judge pattern that keeps emerging; Jacob's contribution is showing that even the planner can often be replaced with deterministic code.

- **Fail-closed is a feature, not a bug** — The benchmark table's most overlooked result: the original skill silently passed broken code; the workflow honestly reported it couldn't fix everything. This is [[Guardrails and Feedback Loops]] in practice: deterministic enforcement catches what prompt-based guardrails miss. A system that says "I'm not done" is safer than one that says "good enough" to save tokens.

- **Versioned, typed data as the coordination substrate** — Jacob stores sub-agent results as versioned, typed data rather than passing raw LLM outputs between agents. This gives full visibility into what each agent produced and avoids re-running expensive agent calls when only one input changed. The [[Swamp Club]] framework provides these primitives (Zod-typed models, immutable versioned data); Jacob is showing them in production, not just in the README.

- **Black box UAT as the trust mechanism** — Instead of reviewing the translated code line-by-line, Jacob had an AI build a test suite that runs both the original and new workflow against the same inputs and compares results. This is black-box validation: you trust the behavior, not the implementation. It's the same philosophy as [[The Oracle Is the Asset]] — the test suite is the durable artifact, the code is disposable — but applied at the workflow level rather than the compiler level.

- **The methodology is the product** — The four-step process (understand → translate plan-first → black-box test → refactor) is more valuable than the specific Swamp implementation. Any deterministic workflow framework ([[Apache Burr]], CI/CD pipelines, [[n8n]] DAGs) could follow the same pattern. The hard part is recognizing which parts of your agent pipeline don't need intelligence.

---

## Critical Analysis

**This is the most practical cost-reduction article in the agentic development canon.** Not because the 8× number is impressive (it is), but because the *methodology* is general. Jacob didn't find a clever prompt or a cheaper model — he found a category error (LLM-as-control-flow) and fixed it with architecture. That's engineering, not tuning.

**The "coordinator agent" pattern needs a name for what it really is: a state machine with a token bill.** Every agent workflow I've seen that loops — "now check X, now decide Y, now dispatch Z" — is spending tokens on transitions that could be a DAG edge. Jacob's 23 → 3 agent reduction isn't about doing less work; it's about moving work from the token meter to the CPU meter. The [[Loop Engineering]] frame is adjacent but orthogonal: Osmani is about designing systems that prompt agents *on a schedule*; Jacob is about designing systems that *don't prompt agents at all* for deterministic steps.

**"Lex Aedificandi, Lex Credendi" is doing a lot of work and I'm not sure Jacob has fully unpacked it.** The idea that architecture shapes belief is true but he applies it narrowly: LLM-in-the-hot-path makes you think intelligence is everywhere. But the deeper version is: *cost shapes architecture shapes belief*. Teams that burn $10K/week on tokens don't just have an architecture problem — they have a feedback-loop problem where the cost signal arrives too late (the credit card bill, not the IDE). The real discipline isn't "use deterministic code" — it's "make token cost visible at the point of workflow design." [[Model Routing Is Simple Until It Isn't]] makes the same point about cache economics; Jacob extends it to architecture-level decisions.

**The fail-open vs fail-closed observation deserves its own article.** Jacob mentions it in passing, but it's the smoking gun for why deterministic orchestration beats LLM coordination on *safety*, not just cost. An LLM coordinator that can say "done" when it isn't done is a systemic failure mode that no amount of prompt engineering fixes — the model wants to be helpful and terminate. Deterministic code that checks preconditions doesn't have that incentive. This is the same argument as [[Guardrails and Feedback Loops]] (linters beat prompts) but applied at the orchestration layer: state machine guards beat coordinator judgment.

**The black-box testing step is under-described but is the secret weapon.** Jacob says "I asked an AI to build a test suite" but doesn't detail what that looked like. A test that can run both the original and new workflow side-by-side requires: (1) a stable interface both can implement, (2) deterministic oracles (what constitutes pass/fail?), and (3) representative test cases. That's the hard part. The AI can generate the boilerplate, but defining "does this code review succeed?" is a judgment call. Jacob's contribution is showing it's *possible* — but replicating it requires domain-specific work he glosses over.

**The Swamp Club framing is both a strength and a limitation.** Swamp provides the primitives (typed models, versioned data, DAG execution) that make this approach natural. But the methodology doesn't require Swamp — any deterministic pipeline (GitHub Actions with explicit steps, a Makefile, a Go program with sequential function calls) can apply the same principle. The risk is that readers interpret this as "install Swamp" when the real message is "find the deterministic program hiding inside your agent workflow and extract it." [[Components of a Coding Agent]] makes the same point about the harness mattering more than the model — but Jacob is showing the *next* step: the harness itself can shed its LLM components as they're understood.

---

## Cross-Links

- [[Swamp Club]] — the workflow framework Jacob uses: Zod-typed models, DAG execution, immutable versioned data
- [[Loop Engineering]] — Addy Osmani's taxonomy of systems that prompt agents; Jacob's approach is the complement: systems that *don't* prompt agents for deterministic steps
- [[The Lifecycle of a Swamp Issue]] — how Swamp enforces state-machine phases that agents can't skip; the same "deterministic guard" philosophy applied to issue tracking
- [[Guardrails and Feedback Loops]] — "linters beat prompts" applied at the orchestration layer: deterministic state machines beat coordinator agents
- [[Agent Orchestration]] — the planner/worker/judge pattern; Jacob shows the planner can often be a DAG, not an agent
- [[The Oracle Is the Asset]] — the test suite as durable artifact; Jacob's black-box UAT approach operationalizes this at the workflow level
- [[Thrifty (Tiered Delegation for Claude Code)]] — cost reduction through tiered model routing; Jacob's approach reduces cost by *eliminating* agent calls, not routing them cheaper
- [[Components of a Coding Agent]] — "the harness matters more than the model"; Jacob shows the harness itself can be progressively de-LLM'd
- [[Agentic Code Review]] — the field report Jacob is responding to; Garfield is a code review tool, and Jacob's rewrite is a case study in making AI code review cheaper
- [[Model Routing Is Simple Until It Isn't]] — cost optimization in production; Jacob's insight that cache economics change routing decisions applies equally to workflow architecture
- [[Code Cleanliness and Coding Agents]] — token consumption reduction through code structure; Jacob's approach is orthogonal: reduce tokens by reducing agent invocations
- [[Specifications as the Product]] — "code is disposable, specs are durable"; Jacob's plan-first translation step applies this to workflow migration

---
*Sources: [[raw/adam-jacob-reducing-token-spend]]*
*Last updated: 2026-07-25*
