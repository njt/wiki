# Building an AI-Native Engineering Org (Gondala)

Sashank Gondala's nine-month field report from FurtherAI: agents did not make the team 5x faster by themselves — reworking every stage of the shipping pipeline did (tests, AI-first review, rules-as-checks, autonomous merging, Slack code channels) — and the resulting bottleneck has migrated out of the code and into planning, review and management, which he proposes to fix by pushing end-to-end ownership onto engineers and redefining what seniority and the EM role mean.

---

The thread opens by answering Will Larson's question about evaluating EMs for AI skills, then widens into an org-design argument. The headline numbers: ~5x merged changes per engineer since January, revert rate under 0.5%, 91% of changes with tests (up from 15%), 99% AI-reviewed before a human sees them, and up to a third of changes merging with no person involved.

---

## The pipeline, stage by stage

The most useful thing in the thread is the sequence. Each stage is dated, which turns it into a maturity model rather than a tool list:

- **Code (March)** — agents write, engineers steer; now "almost 100%".
- **Tests (April)** — 91% of changes include tests, up from 15%.
- **Review (May)** — AI reviews 99% of changes first; humans review the judgment calls.
- **CI (June)** — parallelized, affected-only: 20x the tests in the same 17 minutes.
- **Bug intake (July)** — ClaudeTag in Slack dedupes, files and assigns; 60% of tickets.
- **Rules (August)** — tribal knowledge becomes Python checks that fail the build and print the fix; agents hit them, fix their own code and move on. 21 rules so far; a big reason low-risk changes can merge without a person.
- **Merging (August)** — low-risk changes that pass tests, checks and AI review merge on their own.
- **Code channels (September)** — anyone opens a Slack channel, describes a change, an agent opens a PR; several people steer the same agent in one channel.

> "Every time we sped up one step, work piled up at the next."

This is the queueing-theory view of agent adoption, stated from lived experience: a pipeline is only as fast as its slowest human stage, so optimisation has to be sequential and honest about where the new constraint lands.

> "When a bug or a review comment exposed a rule only engineers knew, we turned it into a small Python check... Agents hit these checks, fix their own code and move on."

Converting review comments into deterministic checks is the highest-leverage move here — it is exactly the review-comment-as-invariant pattern, and it is what makes no-human merging safe rather than reckless.

## The cost, stated plainly

> "None of this was free. Through the summer, much of our engineering effort went into tests and tooling, and customer-facing changes per engineer stayed between 17 and 30 a month."

Five months of tooling investment before customer-facing throughput moved (17–30/month → 45 in September, ~48 in October). Most adoption threads omit this; Gondala makes it the headline.

## The new bottleneck is the org chart

With building compressed, his toy model has planning + review at 70% of feature time, up from 40%. His answer is not more management — it's less:

> "I'm starting to think of engineering seniority in terms of how much of a problem someone can own end to end."

Junior = builds with an agent and can tell when it's wrong. Senior = plans and reviews their own work. Staff = takes a vague problem, makes the tradeoffs, owns the number that says whether it worked. Managers, in turn, must "be in the work" (use features on staging as an end user) and be product-savvy enough that "we shouldn't need a PM or designer to make every small product decision."

## Analysis

This is one of the few sources that reports the *whole* arc — tooling debt, throughput plateau, then the organisational aftershock — rather than just the shiny month. The numbers are self-reported at a small startup and unbenchmarked (no baseline quality comparison beyond the revert rate, and a 0.5% revert rate at a startup says little about a mature codebase). The seniority ladder is the most contestable claim: "junior = can tell when the agent is wrong" quietly assumes the ability to tell, which the comprehension-debt literature suggests is precisely what atrophies when you never write the code yourself.

The strongest idea is the rule-compile loop: every review comment that recurs becomes a deterministic check, which simultaneously raises quality and *expands* what can merge autonomously. That's a ratchet, not a gate — each human review buys permanent autonomy. It deserves to be as canonical as "tests as selection pressure."

The org half is more speculative but more important. Gondala essentially predicts the collapse of the engineer/PM/designer boundary and the EM's conversion from coordinator to product judgment-haver — and admits he doesn't know how it ends: "I suspect this will be a harder problem than automating code, tests, and reviews."

## Relations

- Strengthens [[Two Jobs — Product Engineering and Factory Engineering (Lloyd)]] with a worked example of the factory side: Lloyd's thesis predicts exactly Gondala's summer of tests-and-tooling before product throughput moved.
- Complicates [[The End of Code Review]]: Gondala didn't abolish review, he *relocated* it — AI first at 99% coverage, humans on judgment, and recurring comments compiled into checks that let a third of changes skip humans entirely.
- Nuances [[Code Standards That Survive the Next Agent]]: the 21 Python rules are Bertram's "CI for what machines enforce" taken to its logical endpoint, where the checks' real function is licensing autonomous merges.
- Extends [[Building World-Class Engineering Teams in the Age of AI]] from the platform level to the role level: where Rajan and Dohmke describe bottleneck migration in the abstract, Gondala names who inherits it (engineers own planning and review; EMs become product-savvy end users of the product).

---
*Sources: [[raw/2115832070274723875]], [[summary/2115832070274723875]]*
*Last updated: 2026-11-09*
