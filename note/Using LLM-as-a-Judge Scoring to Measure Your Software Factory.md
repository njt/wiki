# Using LLM-as-a-Judge Scoring to Measure Your Software Factory

Warp's measurement-layer companion to its self-improving-factories argument: grade past agent sessions post hoc with LLM-as-a-judge scorers — full traces in, classifications out — then let observer agents consume the scores and edit the factory's own configuration. Six primitives: trace storage, single-dimension scorers, sampling (≈3% of token spend), a scored corpus, an automated self-improvement loop, and benchmarking.

---

## The argument

Warp frames measurement as the line between guessing and running a factory. The "typical approach" is DORA metrics — PR merge rate, cycle time, defect rate, time to fix errors — which measure external outcomes. The alternative argued for here: score the agent sessions themselves. Because agents leave a complete digital record, every past run is a gradeable artifact, so scoring is retroactive, re-runnable under new rubrics, and cheap to iterate on.

The six primitives, in order:

1. **Trace storage** — not just the conversation but the full input/output envelope: prompts, tool calls, MCP results, images in; PRs, specs, screenshots out. Cloud-stored and API-accessible so scorer agents can load them.
2. **Scoring agents** — one dimension each: task compliance, efficiency, verbosity, code quality, or org-specific checks (did the agent use the right internal MCPs and Skills?). A scorer = judging prompt + classification rubric + judge model. The worked example grades redundant test creation, "a common failure mode we were seeing in our internal factory."
3. **Sampling** — scoring costs money, so not every run gets graded: percent-based or classifier-based samples. Warp's number: ~3% of total token cost.
4. **A scored corpus** — graph scores over time, catch regressions, correlate them with changes to models, skills, and context.
5. **The observer loop** — scorers feed "another agent loop that synthesizes their output in batch and creates updates to the factory definition automatically."
6. **Benchmarking** — scorers double as the grading instrument for comparing model configurations in the factory.

---

## Key quotes

> Since agents provide a complete digital record of their work, you can examine and grade past sessions, see where they are deficient, and adjust going forward.

The enabling claim, and the reason post-hoc scoring differs from a test suite: the trace is already on disk. You can change the rubric and re-grade history without re-running the work. What the piece doesn't say is that this cuts both ways — a rubric change silently rewrites your history, so "the agents got better" and "we redefined good" become hard to distinguish without versioning the rubric itself.

> Each scorer typically focuses on a single dimension like cost or quality, and is defined by a prompt, classification instructions, and a judge model to use.

Deliberately dumb judges, one dimension each — the same decomposition instinct as [[LLM Judge Decision Tree]]'s per-attribute judgments, arrived at from the opposite direction. Single-dimension scorers are auditable ("why did this run fail?" has a small answer space) and composable (the benchmarking layer can weight them per experiment).

> For our internal factory, scoring currently accounts for about 3% of total token costs – that's a reasonable amount to get visibility into agent performance.

The first number in the Warp factory series — [[Self-Improving Software Factories]] was dinged here precisely for shipping zero figures — and a useful one: the observability tax is 3%, declared reasonable without evidence either way. The honest framing is that nobody yet knows the optimal rate; 3% is just where Warp's dial currently sits.

> You can click into the failures and examine what the coding agent did and also examine the scorer run itself, since it's just another agent, to understand why it thinks these coding agent runs produced redundant tests.

The most quietly radical sentence: the judge is itself an inspectable agent, so grading becomes turtles all the way down — agent, judge, human reading both traces. But inspection is not validation. Reading a judge's rationale tells you what it considered, not whether its judgments correlate with what a competent human would say — the distinction the whole technique turns on, glided past.

> Scorers can be input into another agent loop that synthesizes their output in batch and creates updates to the factory definition automatically.

The loop closes here, and the Goodhart engine starts here too. Once the factory's configuration is optimized against scorer output, the scorers stop being measurements and become fitness functions.

---

## Key themes

- **#pattern — Post-hoc trace grading.** The eval substrate is the recorded session, not a fresh benchmark run; the corpus is re-scoreable under new rubrics, which turns your own history into a private benchmark.
- **#pattern — Single-dimension scorers with classification outputs.** One prompt, one dimension, a pass/fail-ish classification. Auditable, composable, cheap enough to sample.
- **#concept — The observability tax.** Scoring costs tokens, so sampling is an economics problem — and classifier-based sampling ("score all my front-end tasks") quietly makes your visibility a non-random subsample.
- **#tool — Warp Factories scoring infrastructure.** Trace storage, scorer definitions, aggregate score store, cron-scheduled scoring agents, self-improvement loop, benchmarking — sold as early access with $10k usage credits.

---

## Critical analysis

The real contribution is plumbing. [[Self-Improving Software Factories]] argued scorers → self-improvement → benchmarks as a taxonomy; this post opens the first box. That's genuinely useful — the redundant-tests scorer is a real failure mode with a real rubric, and 3% is a real number. But the steal-ability is thinner than it looks: the rubric, classifications, and sampling UI in the worked example are screenshots that a text fetch loses, so what survives is the *shape* of a scorer, not a scorer.

The structural weakness is that the judges are never validated, only inspected. Compare [[The Lifecycle of LLM-as-a-Judge]]: Netflix treats a judge as a maintained production component — calibrated against human raters, monitored weekly against a ±2σ band of human disagreement, with a rationale-level meta-judge auditing *why* it fails. Warp's scorers get an inspection ritual where Netflix has a validation protocol. For eyeballed grading of past sessions, that might be fine. But primitive five makes scorers the fitness function for automated factory edits, and at that point unvalidated judges are not a monitoring risk — they're the optimization target. The Goodhart problem the previous Warp piece left unnamed is now structural: the loop Warp sells optimizes the factory against whatever the judge rewards, and nothing in either piece grades the grader.

The DORA contrast is weaker than presented. DORA metrics and judge scores aren't two approaches to the same question — they're outcome measures and process measures. The interesting claim would be that improving the latter moves the former; the piece gestures at it ("try to correlate changes... with improvements") but reports no such correlation. That's the missing experiment: does a factory that optimizes judge scores ship better software, or does it ship agents that please judges? [[Nicole Forsgren on AI and Developer Productivity]]'s first-principles measurement stance is the right standard — measurement that doesn't tie to outcomes is vanity telemetry with extra steps.

Read the vendor frame through, too: the post is an early-access funnel, and the generalizable content lives entirely in the concessions — "if you are building your own you'll want to use some sort of cron-based cloud agent to score prior runs." That sentence is the piece. Everything else is the shortcut.

## Related

- [[Self-Improving Software Factories]] — Strengthens: this is the measurement layer the earlier piece assumed, and it pays the series' first concrete cost number (the 3% observability tax). It also sharpens that note's Goodhart worry into something structural, because self-improvement is now defined entirely as scorer-driven factory edits.
- [[Cloud Software Factories]] — Strengthens: Zach Lloyd's blueprint lists "built-in metrics and evals" as a factory property and moves on; this piece is what building that property concretely requires — full-trace storage, API access, scheduled scorers, an aggregate store — plus the sampling economics the blueprint skipped.
- [[The Lifecycle of LLM-as-a-Judge]] — Complicates: Netflix treats a judge as a lifelong component with calibration, drift monitoring, and rationale-level auditing; Warp deploys judges with an inspection ritual and no validation story. Factory scorers are exactly the kind of judge that lifecycle exists for.
- [[Orchestrating AI Code Review at Scale]] — Parallel: Cloudflare's coordinator judge grades PRs inline as a gate; Warp's scorers grade sessions after the fact as instrumentation. The same LLM-as-judge pattern at two points in the pipeline, with the same open question — what validates the judge?

---
*Sources: [[raw/using-llm-as-a-judge-scoring-to-measure-your-software-factory]], [[summary/using-llm-as-a-judge-scoring-to-measure-your-software-factory]]*
*Last updated: 2026-09-22*
