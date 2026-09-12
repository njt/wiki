# DeepSeek V4 Flash 0731 ARC-AGI Results

DeepSeek V4 Flash 0731 is a reasoning model with three discrete effort levels, submitted to the ARC Prize leaderboard in July 2026. It scores 89.0% on ARC-AGI-1 and 61.4% on ARC-AGI-2 at a cost of $0.02–0.04 per task — one of the most cost-efficient entries on the board. The results are a data point in three ongoing stories: the maturation of reasoning-effort control, the economics of benchmark competition, and the evolving open-weight frontier.

---

## Key Quotes

> At max effort, DeepSeek V4 Flash 0731 scores 89.0% on ARC-AGI-1 Semi-Private at $0.02 per task and 61.4% on ARC-AGI-2 Semi-Private at $0.04 per task.

The headline numbers. ARC-AGI-1 is effectively saturated territory — 89.0% means the model solves nearly 9 in 10 of the hardest remaining puzzles on a benchmark where human baseline is ~80%. ARC-AGI-2 at 61.4% is the more informative score: it's a harder benchmark, less contaminated, and the gap between Max and Low (15.4pp) tells you reasoning depth is doing real work here, not just adding token padding.

> The gap between Max and Low is 5pp on ARC-AGI-1 but 15.4pp on ARC-AGI-2.

Not quoted directly from the page but derived from the verified scores table. This asymmetry is the most interesting structural finding: on the easier benchmark, reasoning effort barely moves the needle. On the harder one, it moves it enormously. This is exactly what you'd predict from the reasoning-effort design space [[Controlling Reasoning Effort in LLMs]] maps — but it's rare to see it quantified this cleanly in a single model family on a single leaderboard.

> No ARC-AGI-3 scores reported.

The absence is loud. ARC-AGI-3 is the hardest tier, and DeepSeek either didn't run it or didn't publish. Given the $0.02–0.04/task cost and the model's efficiency profile, the former seems more likely — this is a Flash variant, optimized for speed and cost, and ARC-AGI-3 may simply not be the target.

---

## Key Themes

- #concept **Reasoning-effort monotonicity** — Across all 520 public eval tasks (400 ARC-AGI-1 + 120 ARC-AGI-2), there's no case where a lower effort level passes but a higher one fails. This isn't guaranteed — reasoning models can overthink simple problems — but the monotonicity holds perfectly here. The mechanism likely matters: DeepSeek V4's approach to reasoning control (per [[Controlling Reasoning Effort in LLMs]]) uses separate specialists with different context windows rather than a single model with a length-penalty knob, which may avoid the overthinking failure mode.
- #concept **Benchmark difficulty as reasoning amplifier** — The ARC-AGI-1 gap (5pp) vs. ARC-AGI-2 gap (15.4pp) quantifies something usually left implicit: reasoning depth matters more on harder problems. This is the empirical mirror of the theoretical claim in [[The Reasoning Trap]] — if every reasoning enhancement amplifies both capability and hallucination, then the capability gain should be largest where the problem is hardest, and the hallucination cost should be invisible on a benchmark with no tool-use component.
- #concept **Cost efficiency as benchmark signal** — $0.02/task puts DeepSeek V4 Flash at a radically different price point than frontier closed models. At these costs, benchmark scores are no longer just capability signals — they're economic signals. A model that scores 89% at $0.02/task may be more practically useful than one that scores 92% at $2.00/task, especially for the high-volume agent loops where ARC-style reasoning capability is embedded in larger workflows.
- #tool **ARC-AGI as evaluation tier** — The ARC Prize leaderboard uses Semi-Private evaluation, which is more resistant to contamination than fully public benchmarks but less so than fully private ones. [[How Far Behind Are Open Models]] flags ARC-AGI's semi-private contamination as a methodological concern — the scores are more trustworthy than public leaderboard numbers but less trustworthy than a custom private eval.

---

## Critical Analysis

**This is a thin source — a leaderboard entry, not an article — but it's a useful data point.** The page is essentially a scorecard: verified scores, per-task pass/fail matrices, and cost figures. There's no methodology discussion, no system card, no comparison to other models. The value is in the numbers and what they imply, not in what the page says.

**The cost numbers are the most important and least-discussed aspect.** $0.02–0.04 per task is extraordinarily cheap for a reasoning model. For context, the inference economics described in [[Inference Cost Napkin Math]] and [[GPU Self-Hosting for Coding Agents]] suggest these costs reflect either aggressive optimization (DeepSeek V4's architectural innovations like CSA/HCA, per [[Recent Developments in LLM Architectures]]) or subsidized pricing. Either way, the economic signal matters more than the benchmark signal for anyone building systems that call reasoning models at scale.

**The ARC-AGI-2 gap is the story worth watching.** 61.4% Max vs. 46.0% Low — a 15.4pp swing from reasoning effort alone. This is the cleanest public demonstration of [[Controlling Reasoning Effort in LLMs]]'s central claim: effort control isn't a feature, it's an economic lever. If you're running DeepSeek V4 Flash in an agent loop, you'd use Low for most calls (saving >50% on cost, assuming effort scales roughly with cost) and reserve Max for the hardest sub-problems. The architecture of that routing decision — when to spend the tokens — is the same problem [[Model Routing Is Simple Until It Isn't]] diagnoses.

**The missing ARC-AGI-3 score is conspicuous.** ARC-AGI-3 is designed to be the hardest tier, still far from saturation. DeepSeek's absence there might mean the Flash variant wasn't designed for it, or it might mean the score wasn't competitive enough to publish. Either way, the pattern fits the [[Goodhart's Law and AI Benchmarks]] thesis: labs publish the scores that tell the story they want told, and the missing numbers are as informative as the present ones.

**The reasoning-effort monotonicity is a non-trivial finding.** Most reasoning-effort discussions focus on the cost/quality trade-off, but the structural question — can more reasoning ever *hurt*? — is largely unanswered. [[The Reasoning Trap]] shows that reasoning amplifies hallucination, but ARC-AGI tasks have no tool-use component, so the hallucination cost is invisible here. The monotonicity might not hold on benchmarks that include tool calls or where overthinking could send the model down wrong paths. This is a gap in the evidence the leaderboard page doesn't address.

**Where this fits in the wiki:** This is a concrete data point that ties together several threads. The reasoning-effort story in [[Controlling Reasoning Effort in LLMs]] gets an empirical case study. The benchmark-integrity story in [[Goodhart's Law and AI Benchmarks]] gets a reminder that public scores are marketing artifacts with missing rows. The open-weight story in [[How Far Behind Are Open Models]] and [[GLM-5.2 Is the Step Change for Open Agents]] gets another Chinese lab shipping competitive results at aggressive prices. And [[The Reasoning Trap]] gets a natural experiment — reasoning depth helps most on hard problems, but the hallucination cost is invisible on a benchmark that doesn't test for it.

---

*Sources: [[raw/deepseek-v4-flash-0731]], [[summary/deepseek-v4-flash-0731]]*
*Last updated: 2026-08-08*
