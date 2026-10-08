# Kantar LINK AI Content Optimizer

A Microsoft Frontier Company case study on building Kantar's LINK AI Content Optimizer: an agentic learning loop that converts three decades of ad-effectiveness data from a reporting signal into an optimization engine — score, recommend, generate, re-evaluate — with the honest admission that every component inside the loop is itself an optimization problem.

---

## What it argues

Kantar built its franchise on a question — "will this ad work?" — answered by 35 years of consumer testing: 35 million interactions, 260,000+ ads. LINK AI turned that into ML scoring in minutes. But customers increasingly want not a score but an *improved asset*, which meant building an agentic system grounded in Kantar's rubrics: rubrics become graders, a reasoning agent with packaged skills produces recommendations and regenerated assets, and the same graders measure whether the asset actually improved.

The architecture is loops inside loops. The outer loop improves the creative. Inner loops improve the graders, skills, recommendation logic, orchestration, and evaluation process — driven by SME feedback, automated evals, and traces.

## Key quotes

> "Write the rubrics first. Turn rubrics into graders and expect to need more than one level. A whole-asset score locates you. A component-level score tells you what to change. Make grading fast. If a turn is too slow, there is no loop, only a report."

The most quotable distillation of the whole piece. The insight that a whole-asset score "locates you" but can't steer you is the same granularity argument that recurs across agentic evaluation: decomposed metrics are what make a feedback loop *actionable* rather than merely observant.

> "Neither kind of eval is sufficient alone. Automated judging scales and drifts. Expert judgment is the calibration and doesn't scale. Running both and using each to check the other is critical."

A crisp statement of the paired-eval doctrine. Notably they use LLM-based judging to *read across SME comments* — automating the meta-review, not replacing the reviewers — and use expert labels to recalibrate the judges over time.

> "A tunable harness creates possible changes. Evaluations decide which of them are improvements. Without the second half, the first half is a churn."

The cleanest one-sentence version of the harness-eval dependency that runs through this wiki.

> "A competitor can use the same foundation model. It can adopt similar infrastructure. It can draw the same architectural diagram. What is hard to reproduce is a scoring system built on decades of proprietary data with rubrics engineered to scale, graders calibrated by people who know the domain, and an experimentation process that converts that evidence into a better system."

A moat argument for the agentic era: the model and the diagram are commodities; the evaluated loop over proprietary data is not.

## Notable engineering details

- **Grading granularity**: six dimensions, extended to scene- and image-level scoring, because a six-second fix in a 30-second video is diluted in the whole-asset score. Scene-level grading "exposes the undiluted change" — a tighter signal for both recommendation and evaluation.
- **Skills as governance, not isolation**: Kantar's proprietary knowledge lives in versioned skill components rather than the agent's general instructions; the value claimed is modularity, reuse, and governance — with a nod that stronger isolation would demand sub-agents or separate execution contexts.
- **Latency as product requirement**: repeated grading turns latency into a product feature. vLLM migration (~8x), a single ML service template (ship in hours instead of weeks — notably, for the team *and its coding agents*), Dapr Workflow orchestration of 20 services, and prioritized queues taking a first load test from 80% failure to 95% pass.

## Take

This is one of the better-written vendor case studies because it names the uncomfortable economics: expert evaluation is expensive, needs its own tooling, and produces more feedback "than anyone can hold in their head" — which is precisely the problem automated judging exists to absorb, and precisely why it can't be trusted alone. The self-criticism (first load test failing 80% of requests) gives the infrastructure claims credibility.

The moat paragraph is the strategic punchline, and it's correct as far as it goes: models and diagrams are copyable, calibrated proprietary graders aren't. But note what it quietly assumes — that the *ground truth* (LINK AI's predictive scores) is actually valid. The whole loop optimizes against Kantar's own grading signal; if that signal has blind spots, the loop will faithfully optimize into them. Eval loops are only as honest as the grader at the bottom, and here the grader is itself a model ensemble. The paired SME/judge setup is the hedge, but the circularity is structural.

#concept #tool #pattern

## Relations

- Strengthens [[Eval-Driven Development (Airbnb)]]: Airbnb's three-layer toolkit (programmatic → LLM-as-judge → human) is exactly the structure Kantar landed on from the product side, and adds the concrete calibration mechanism — expert comments judged *by an LLM meta-reviewer* — for the top layer checking the middle one.
- Nuances [[The Lifecycle of LLM-as-a-Judge]]: Netflix tunes one judge against human raters in production; Kantar shows the same pairing working pre-production, where expert evals calibrate the judge before any drift-monitoring can exist.
- Complicates [[Building Shippy — Agent Architecture for High-Stakes Domains]]: both decompose a domain's proprietary judgment into skills plus graders, but Shippy leans on deterministic CLI wrappers for verification while Kantar's grader-of-last-resort is itself an ML ensemble — the circularity Shippy avoids by design.
- A production-scale instance of [[Shipping AI Agents to Production]]'s thesis that context, not the model, is the durable advantage: the moat here is literally 35 years of proprietary grading data wrapped in a learning loop.

---

*Sources: [[raw/kantar-link-ai-content-optimizer-advertising-learning-loop]], [[summary/kantar-link-ai-content-optimizer-advertising-learning-loop]]*
*Last updated: 2026-10-08*
