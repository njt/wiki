---
url: https://blog.watson-labs.co.uk/6-learnings-from-12000-agentic-code-reviews/
date_fetched: 2026-09-13
---

## The setup

Gymwasp is a Swamp customer, currently in stealth. I helped them get their software factory up and running: the pipeline that orchestrates code planning through to shipped code, agentically using swamp as the harness. No human is reading any code. They've been generous enough to share the metrics that swamp collected on their factory's performance across 372 issues over the span of 6 months, and those metrics are what this post is built on.

Here's what the data shows:

- **65% of issues are merge-ready after a single agentic review round.**Most of the value lands immediately.
- **After round 4, extra rounds introduce as much uncertainty as they're trying to fix.**The conditional pass rate halves after round 2 and flatlines around 87% cumulative. The loop oscillates rather than converges.- *(this is exactly what we see in our internal factory too)*
- **29% of issues never reached a clean pass.**They shipped on warn. This is their measure of risk tolerance.
- **The naive average understates cost by 44%.**If you only count the issues that clean passed, you get 3.31 rounds. The honest number, when including issues that shipped with warnings, is 5.99.
- **Per-issue review cost doesn't scale with volume.**Shipping volume tripled in one month. The median held at 4 rounds, a result of using a deterministic harness.
- **1,801 adversarial review rounds: 1,463 on code, 338 on plans.**The agent is far more likely to write the code wrong than write the plan wrong.

## The factory

They call their factory **The Mandible**, which correlates well with the factory we run internally for Swamp itself:

- **Plan**gathers data, then qualifies the scope.
- **Build**produces code.
- **Review**catches what doesn't meet the spec or the quality criteria.
- **Ship**closes the loop.
- **Rework**arcs route failures back in the flow.

A human is in the loop at Plan and Review, but they're not reading code. They're asserting whether the agents understood the problem correctly and making product calls the agents can't make for themselves.

## How The Mandible works

At Plan and Review, all 7 reviewers run in parallel against either the plan or the written code. Each one is a separate agent process on its own model instance, so none can see what the others found. Each reviewer returns a verdict:

- **pass**no concerns in that lane.
- **warn**wants follow-up, doesn't block the merge.
- **fail**blocks shipping until fixed.

The round's overall verdict is the worst of the seven.

The seven lanes:

- **Test coverage.**Assumes code is broken until a real-fixture integration test proves otherwise. Strictest lane by design.
- **Clean code.**Scope creep, dead-on-arrival code, premature abstraction, hardcoded values outside the registry.
- **Frontend.**Component structure, design tokens, responsiveness.
- **DDD.**Bounded contexts, aggregate boundaries, layer separation.
- **Security.**Authn, authz, injection, IDOR, race-on-auth. No skill file, purely adversarial.
- **Accessibility.**Keyboard, ARIA, contrast, hit targets. WCAG 2.1 AA.
- **Observability.**Spans, web events, error propagation.

Each reviewer gets a narrow brief *and* an explicit out-of-lane exclusion list. Without exclusion lists, you get the same finding seven times.

## The numbers

Most teams aren't measuring agentic review at all. The few that do are likely measuring either the time it takes to get a change out, or the magnitude of issues they can get through in a timeframe. Neither are useful when looking at optimising the process.

Here's what was tracked, and what it can tell you. The chart below shows the probability an issue has earned a clean pass (all 7 ✅) by review round N.

i. Anthropic Claude Opus was used throughout, moving from 4.6 to 5 as releases landed during the measurement window.

ii. The power of averages here assumes that with a dataset of 372 issues over 6 months, the "average" PR is representative over time.

iii. Every software factory is tailored to its own organisation and needs. Results and findings will vary based on implementation.

Thanks to the absolute legend James Owens for sharing the data (best man at my wedding too, so top lad).

## How to adjust when it stops working

The data doesn't just tell you whether the process is working *now*. It tells you when it's drifting.

- 
**Pass rate per round stops improving**means your reviewers are oscillating, not converging. The fix isn't more rounds. It's changing the briefs. Tighten the exclusion lists. Sharpen the failure vocabulary. Make each lane's definition of*fail*more concrete so the threshold doesn't wobble between rounds.
- 
**One lane dominating the fails**means either that lane is genuinely catching more problems (good) or its brief is tuned too aggressively (bad). Test coverage at 19.1% is by design, its brief*orders*it to fail when an integration test is absent. If your observability lane suddenly starts blocking at 15%, something changed, and it probably isn't the code quality.
- 
**Infrastructure flakes eroding trust**is the quiet killer. 3.7% of The Mandible's verdicts were lost to agent crashes. 8.1% of rounds had at least one lane unassessed. 111 iterations died before writing a verdict. Go check how many of your test flakes died because of CPU pinning or OOMs in your CI runners. Same concept, different domain. In classic CI, when the flake rate climbs the temptation is to comment the tests out because they're providing no value. Here, if your clean code reviewer is failing frequently, the temptation is to remove it. Same instinct, same mistake. Fix the brief, don't kill the lane.
- 
**The "Mergeable" criteria requiring more rounds**means your loop is getting worse. Round 4 is the elbow in this data, the point where extra rounds stop buying quality. If next month it's round 6, the briefs have drifted, the exclusion lists are stale, or the codebase has moved enough that the anchors are pointing at the wrong files. Same problem, same fix: instrument, measure, retune.
- 
**Perfection is a fallacy.**29% of issues shipped without a clean pass. The pipeline caught the structural problems in the first three rounds, and the warns logged everything else. Chasing 100% on a complex PR given the non-determinism of LLM-based reviews is not a productive use of compute. Ship at merge-ready, batch the follow-ups.

## The same discipline, new domain

I've written and managed literally thousands of CI/CD pipelines across hundreds of production environments. This is CI/CD optimisation pointed at a new target. The mental model transfers because the shape transfers: a deterministic pipeline with measurable stages, each of which can be instrumented, analysed, and improved.

- Instrument the pipeline. Track rounds, verdicts, per-lane breakdowns.
- Measure honestly. Survival analysis, not naive averages. The censored observations are data, not noise.
- Find the elbow. Which round does extra work stop paying? Which lane blocks most? Where's the churn?
- Retune. Tighten briefs, sharpen exclusion lists, adjust the threshold between warn and fail.
- Measure again. Did the hazard rate improve? Did the elbow shift left? Did the flake rate drop?

The old discipline was making builds fast and tests reliable. The new discipline is making review accurate and review loops short. The techniques transfer. The tools MUST be different.

That's the project. Every number in this post came out of swamp's data plane as a byproduct of the work, not as archaeology performed afterwards. Swamp is deterministic automation for AI agents: the scaffolding that controls what each reviewer sees, tracks every verdict, and gives you the data to know whether your loop is converging or just spinning. Check it out at swamp-club.com.
