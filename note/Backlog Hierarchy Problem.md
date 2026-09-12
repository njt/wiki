# Backlog Hierarchy Problem

ProdPad's argument that the flat backlog is a structural failure: when "replace the payment system" and "add a button to this page" compete as peers in the same list, prioritization breaks before any framework is applied. The fix is hierarchy — a three-tier structure (objectives → initiatives → ideas) borrowed from Klaus Leopold's flight levels, plus a two-backlog split between discovery and delivery — so that scoring happens within a level instead of across incommensurable altitudes.

---

## Key Quotes

> "The word 'idea' is doing too much work, and nobody in the room realizes it."

The diagnosis in one line. A single bucket that holds a multi-quarter platform bet, a CSS tweak, and a sentence pasted from Slack can't support a comparison. The flat list is the bug; the squishy vocabulary is just its symptom.

> "a graveyard with a search bar."

The Head of Product's description of their 400-item backlog. It names the actual experience: the list isn't a strategy tool anymore, it's a place dead ideas go to be searched, not decided.

> "Both are purchases. Both cost money. The comparison is still absurd."

The house-vs-lunch analogy for mixed-altitude ranking. Two items sharing a category word ("idea," "purchase") doesn't make them comparable. The deeper claim: comparability is a property of the *structure*, not of the items.

> "The math produces a number. The number creates an illusion of rigor. But the inputs are incommensurable, so the output is fiction dressed up as analysis."

The scoring trap, and the most broadly useful point. RICE and Impact-vs-Effort fail not because the frameworks are bad but because they assume the scored items are comparable. "Reach" for a platform migration and "Reach" for a tooltip measure different things, over different horizons, for different users. Scoring without hierarchy produces either false precision or abandoned scoring — both from the same root cause.

> "This is a good idea, but it doesn't serve any of our current initiatives."

Why the hierarchy makes "no" easier. When every idea exists in isolation, rejecting one feels personal. When ideas sit inside initiatives that connect to objectives, the refusal has structural support — the hierarchy absorbs the conflict that would otherwise land on the PM's judgment alone.

> "A backlog is not a to-do list. A to-do list is a flat sequence of actions. A backlog is a decision system."

The closing thesis. A to-do list doesn't need hierarchy; a decision system does, because it exists to surface trade-offs and make commitments at the right altitude. Strip the hierarchy and you get a wishlist, not a system.

---

## Key Themes

- **#concept Flight levels** — Klaus Leopold's model (strategic / coordination / operational) as the theoretical spine. Each altitude has its own cadence, decision-makers, and definition of "done." A single-altitude backlog drags strategy into operational detail and inflates button debates into strategic ones.
- **#concept The flat backlog trap** — A flat list implies every item is the same type of thing deserving the same evaluation. True at 20 items, unmanageable at 200, a liability past 500.
- **#concept Three-tier hierarchy** — Objectives ("where are we going," quarterly, leadership-owned), initiatives ("what problem, what success looks like," the coordination layer), ideas ("how might we solve it," always evaluated in the context of a parent initiative).
- **#pattern Two-backlog principle** — A product/opportunity backlog (ideas, experiments, hypotheses, organized around problems) upstream of a delivery backlog (committed work, ready stories). Marty Cagan's opportunity-vs-delivery split, restated at the backlog level.
- **#pattern Score within a level** — The cure for the scoring trap. Compare initiatives on strategic criteria, ideas on tactical criteria (feasibility, speed to learn). Like-for-like comparison is what makes the number meaningful.
- **#person Klaus Leopold** — Flight-levels originator; his core insight (different altitudes need different decisions and feedback loops) is what the article repurposes for backlogs.

---

## Critical Analysis

The diagnosis is right and, for a vendor blog, refreshingly vendor-agnostic. The core claim — that prioritization frameworks fail in flat backlogs because they assume comparability — is the kind of structural insight that transfers cleanly out of product management into any domain where mixed-altitude work gets ranked in one list. The "score within a level" prescription is the correct fix, and it's more honest than the "just use this framework" advice it's implicitly correcting.

But the article is a ProdPad piece, and its fix is self-serving in a way worth naming. The three-level hierarchy maps almost one-to-one onto ProdPad's own data model (objectives at the top, initiatives as Now-Next-Later cards, ideas underneath), and the "delivery tools make this worse" section exists to position ProdPad as the upstream corrective. None of that makes the argument wrong — a tool company writing about the problem its tool solves can still be correct — but the reader should notice that the remedy always terminates in "adopt the structure ProdPad imposes."

The deeper gap: the article assumes teams *can* restructure their backlog, and ignores the incentive problem that keeps it flat in the first place. A flat backlog is not an accident of tooling; it's a side effect of what's cheap to record (a ticket anyone can add) versus what's expensive to decide (whether something deserves its own initiative). The [[Discovery Debt]] critique applies here in mirror image: you can't fix a structure problem with a time-allocation or tooling policy when the forces producing flatness — sales asks, stakeholder noise, career incentives tied to shipping — are the same forces that will flatten your new hierarchy next quarter. The initiative layer is the *easier* half of the fix; the hard half is defending it.

There's also a loose joint in the flight-levels mapping. Leopold's three levels are organizational altitudes — portfolio, coordination, operations — spanning *teams*, not a single backlog. The article compresses them into one team's three tiers and quietly demotes the coordination layer (cross-team dependencies, initiative-level trade-offs) into an "initiative" card. It works as a metaphor, but it flattens exactly the layer the model was built to foreground.

What the article misses by predating AI is what makes it urgent *for this wiki*. The flat backlog is an output-optimizing structure; AI is an output multiplier. [[The AI Productivity Paradox]] argues AI speeds up a broken project model, and a flat backlog is that broken model's native habitat — the place where "build more" is the only legible move and "should we build it" has no column. The two-backlog principle is, in effect, the structural precondition Cagan's discovery/delivery split needs at the backlog level: without it, AI just generates mixed-altitude ideas faster than a flat list can possibly sort them.

---

## Related Pages

- [[The AI Productivity Paradox]] — Cagan's discovery/delivery split; this article is the backlog-level operationalization of it, via the two-backlog principle
- [[Discovery Debt]] — Untested assumptions compound invisibly in a flat backlog, where discovery work can't be told apart from delivery work
- [[Zero Alignment]] — "Should we build it" vs "how to build it"; hierarchy is the structural answer to the alignment bottleneck
- [[Hidden Inefficiencies Behind Delivery Delays]] — Priority churn, one of El-Deeb's five invisible delivery killers, traces upstream to the flat backlog's recency bias
- [[AI for Product Management]] — The PM practice layer; where the "what matters" conversation this hierarchy structures actually happens
- [[Dev Machine Foundry]] — Priority inversion: measurable near-term work crowds out strategic bets, the exact failure a flat backlog encodes

---

*Sources: [[raw/backlog-hierarchy-problem]], [[summary/backlog-hierarchy-problem]]*
*Last updated: 2026-09-08*
