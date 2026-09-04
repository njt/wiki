# Eval-Driven Development (Airbnb)

Airbnb's production-hardened playbook for evaluating LLM-powered features at scale: eval-driven development as the GenAI analogue of TDD, a three-layer evaluation toolkit, the calibration methodology that makes virtual judges trustworthy, and the specific challenges of evaluating agentic systems where trajectories matter as much as final outputs.

---

## Key Quotes

> "Expect to spend a meaningful share of your total project effort on evaluation. This is not unnecessary overhead, it's how you build products that actually work."

This is the thesis statement. Airbnb is arguing that evaluation *is* the development work, not a separate QA phase. The framing echoes [[LLM Evals]]'s claim that evals consume 60–80% of your time — but Airbnb goes further by embedding it in the engineering workflow rather than treating it as a cost center.

> "When in doubt, look at your data. Manually reviewing your data and building an intuition for what counts as success is always the starting point we recommend to teams."

The "one rule" that precedes all methodology. Build your prototype, run it through 100 examples, read every output, categorize the failures, then build evals. This is convergent with Hamel Husain's "error analysis is the most important activity in evals" — but Airbnb frames it as a disciplined habit rather than a diagnostic step.

> "A virtual judge that hasn't been calibrated is worse than no judge at all, because it gives you false confidence."

The hard-won production insight. Airbnb prescribes a specific calibration loop: golden dataset (50–100 examples, must include failures) → run judge → measure agreement (Cohen's kappa or Krippendorff's alpha, target high 80s–90s%) → analyze disagreements → refine rubric and few-shot examples → repeat. This is the most detailed public calibration recipe from a major engineering org.

> "3–5 well-calibrated LLM-as-judge evaluators beat 20–30 noisy ones. Each should target one specific correctness dimension."

The "one evaluator per dimension" rule. No God evaluators. This cuts against the instinct to write a single comprehensive eval prompt and validates [[LLM Evals]]'s preference for binary pass/fail over Likert scales — each dimension gets its own sharp yes/no.

> "Evaluating only the final output is insufficient: a correct final answer can mask a broken reasoning path, wrong tool parameters, or an inefficient trajectory."

The agentic evaluation thesis. Airbnb recommends three layers for agentic systems: final output quality, trajectory correctness (right tools, right order, right parameters), and intermediate state validity. This extends [[Demystifying Evals for AI Agents]]'s "grade outcomes, not pathways" — Airbnb says grade both, but separately.

## Key Themes

#evaluation #testing #LLM-as-judge #calibration #agentic-systems #production-engineering #methodology #EDD

## The EDD Framework

Five principles anchor eval-driven development:

1. **Define goals and gates upfront.** What are you optimizing for? What must be true before you ship? These may emerge from data exploration rather than being known at the start.
2. **Let real errors guide your metrics.** Co-develop evals with cross-functional partners based on observed failures, not hypothetical ones.
3. **Keep evaluators small and sharp.** 3–5 well-calibrated judges, each targeting one correctness dimension.
4. **Appoint a decision-maker.** Team discussion shapes correctness, but a single human makes the final call on good vs. bad.
5. **Collaborate continuously.** Product partners regularly answer: "Is X better than Y?" and "What's actually wrong with this output?"

This is more structured than the "benevolent dictator" pattern from [[LLM Evals]] — Airbnb makes the decision-maker role explicit and formal, and adds the continuous product collaboration as a named principle.

## The Three Evaluation Layers

**Layer 1: Programmatic checks.** Deterministic, code-based, catches obvious failures (JSON validity, length bounds, format correctness). Use structured outputs with JSON schemas — don't rely on prompt instructions alone to enforce data formats.

**Layer 2: LLM-as-Judge.** A stronger model evaluates output against a carefully designed rubric. Ambiguity is the enemy: "if a human can't apply the rubric consistently, an LLM certainly can't." The article includes a worked example rubric for readability scoring with explicit 0/1 criteria.

**Layer 3: Human evaluation.** Gold standard for ground truth, high-stakes domains, and resolving disagreements between automated evaluators. The rule of thumb: start with 20–100 SME-labeled rows, move to scaled annotation only when the rubric is rock-solid.

This three-layer model maps cleanly onto [[Demystifying Evals for AI Agents]]'s three grader types (code-based, model-based, human) — convergent evolution from different engineering orgs.

## Calibration: The Missing Step Most Teams Skip

The calibration loop is the article's most detailed contribution:

1. Create a golden dataset of 50–100 examples (must include bad ones — you can't test discernment without them)
2. Run virtual judge against the golden set
3. Measure agreement (Cohen's kappa or Krippendorff's alpha, target high 80s–90s%)
4. Analyze disagreements, refine rubric and few-shot examples
5. Recalibrate periodically as failure modes evolve

The article gives a concrete example: a faithfulness judge initially agrees with the PM 78% of the time, penalizing accurate paraphrases as "unfaithful." After rubric refinement and few-shot examples, agreement jumps to 88%.

This is the practical answer to [[Prompt Debt]]'s "measurement, not prose" imperative — calibration turns a natural-language rubric into a measurement instrument.

## Evaluating Agentic Systems

The article's agentic evaluation framework addresses a gap in most eval guides. For agentic systems with multi-step reasoning, tool calling, and branching logic:

- **Don't just evaluate the final answer.** A correct output can mask wrong tool parameters or an inefficient trajectory.
- **Use trace data.** Agent traces (spans under an application root) contain agent type, subagent invocations, tool calls, and I/O.
- **Reconstruct traces via DFS.** Traverse the trace tree to verify correct subagent invocation, tool selection, and step sequencing.
- **Scope evaluation to specific agents/subagents.** Don't evaluate the whole system as a monolith.

## Practical Walkthrough

The article's end-to-end example (a travel support policy assistant) demonstrates:

1. **Discover**: Run 100 inputs, categorize failures (hallucination, verbosity, over-refusal, format)
2. **Build**: Programmatic checks for JSON/length + virtual judges for faithfulness and conciseness + PM-labeled golden set of 60
3. **Calibrate**: Iterate rubric until judge agrees with PM at 88%
4. **Scale**: Run across 5,000 examples, set up production monitoring with 5% daily sampling and weekly PM review

The meta-lesson: **fix one variable at a time.** Fix model, vary prompt → fix prompt, vary model → fix both, vary serving config. Evaluators and candidates sharpen each other until both stabilize.

## Critical Analysis

**Strong.** The calibration methodology is the most detailed public recipe from a large engineering org. The "one evaluator per dimension" rule and the worked rubric example make abstract advice concrete. The agentic trajectory evaluation framework fills a gap — most eval guides stop at output quality. The "fix one variable at a time" discipline is underappreciated and would prevent weeks of thrashing on most teams.

**Tensions with existing advice.** Hamel Husain ([[LLM Evals]]) explicitly advises *against* eval-driven development — writing evaluators before implementation — arguing you need to see real failures before you can write meaningful tests. Airbnb embraces EDD as the formalized version of exactly that: discover failures from data, then encode them as evals. The disagreement may be semantic: both agree you shouldn't write evals in a vacuum, but differ on whether the TDD framing helps or misleads.

**Missing.** The article underplays cost. Calibrating 3–5 virtual judges with golden datasets of 50–100 examples each is real engineering work, and the "meaningful share of project effort" on evaluation could easily become the majority share. For a team shipping their first LLM feature, this level of rigor may be aspirational. The article also doesn't address eval saturation — how do you know when your evals have stopped catching new failure modes? The connection to [[Benchmark Exploitation]]'s demonstration that evals can be gamed to 100% is left unexplored.

**The "decision-maker" principle is double-edged.** Explicitly naming a final human arbiter prevents deadlock, but it also concentrates quality judgment in one person. If that person leaves or burns out, the eval infrastructure may lose calibration. The article doesn't discuss succession or rotation.

**The agentic eval framework is promising but underdeveloped.** DFS traversal of trace trees makes sense, but the article doesn't address how to handle non-deterministic agent paths — if an agent takes a valid but unexpected route, does the trajectory eval flag it as a failure? This is the same tension described in [[Demystifying Evals for AI Agents]]'s warning against overly rigid step-checking.

## Connections

- [[LLM Evals]] — Convergent on error analysis as the foundation, but diverges on EDD framing. Airbnb adds calibration methodology.
- [[Demystifying Evals for AI Agents]] — Convergent on three-layer grader taxonomy and "start small." Airbnb extends with agentic trajectory evaluation.
- [[Prompt Debt]] — EDD is the practical implementation of "measurement, not prose": calibration turns rubrics into measurement instruments.
- [[Guardrails and Feedback Loops]] — Airbnb's three layers map onto the enforcement hierarchy, adding the calibration step that makes the LLM-as-judge layer trustworthy.
- [[Goodhart's Law and AI Benchmarks]] — The calibration loop is a defense against Goodhart: when your judge is calibrated to human judgment, it's harder to game.
- [[A New Era for Software Testing]] — Airbnb's "look at your data" principle is antirez's checklist-driven QA applied at the evaluation layer.
- [[The PM's Playbook for Shipping AI Features]] — Airbnb's "collaborate continuously" principle operationalizes Gaurav Savla's quality pyramids with a specific cross-functional workflow.
- [[Sidekick's Continual Learning Loop]] — Shopify's production flywheel converges on the same calibration recipe (rubric, blind annotation, Cohen's kappa) with two twists worth stealing: 25 *random* samples rather than a curated golden set, and treating human inter-annotator agreement as the judge's ceiling rather than a target to exceed.
- [[Flexible Authentication (Airbnb)]] — The same company's server-driven auth rebuild is the *other* half of the iteration culture EDD describes: server-driven screens bought the 20+ experiments in three months, and EDD's eval discipline is what measures whether those experiments actually improved login. One move creates velocity, the other makes it trustworthy.
- [[The Lifecycle of LLM-as-a-Judge]] — Netflix extends Airbnb's calibration recipe into a full production lifecycle: the same rubric-plus-rationale benchmark, but then *deploys* the judge as both gate and revision critic, and *monitors* it with a weekly 300-example human-in-the-loop drift check (a ±2σ band below the average rater) rather than re-calibrating only periodically.
- [[Shipping AI Agents to Production]] — the enterprise (AWS/Arize sponsored) restatement of the same three-layer recipe, with one twist worth keeping: production traces, failures included, as the golden dataset rather than a curated set — and the claim that context, not the model, is the durable advantage.
- [[Analytical AI]] — Sutro's category names what EDD is actually engineering: evals are a special case of analytical AI (a judge deciding whether output is correct is a decision task, not a creative one), which is why the whole apparatus — ground truth, calibration, discriminative scoring — inherits analytical AI's measurability and its permission to use smaller, evaluated models.
- [[AI Team Mistakes]] — Doug Turnbull supplies the search-side slogan behind EDD: great AI orgs spend ~50% of investment on *understanding* the problem, not solving it. His "actively NOT trust my instincts" is Airbnb's "look at your data" as a personal discipline rather than a team methodology — convergent on evals-first, with EDD providing the calibration recipe Turnbull only gestures at.

---
*Sources: [[raw/eval-driven-development-lessons-from-evaluating-genai-at-scale-e817e5ae5788]], [[summary/eval-driven-development-lessons-from-evaluating-genai-at-scale-e817e5ae5788]]*
*Last updated: 2026-08-07*
