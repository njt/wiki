---
url: https://www.oreilly.com/radar/what-a-user-story-actually-costs-in-a-dark-code-factory/
title: What a User Story Actually Costs in a Dark Code Factory
site: O'Reilly Radar
author: fxmartin
date_published: 2026-07
date_fetched: 2026-09-04
topics:
  - agent-coding-workflow
---

fxmartin built a production application of 861,601 lines — 696 user stories and 779 merged PRs over 105 days — but couldn't say what it cost. The first-generation autonomous SDLC framework driving Claude Code kept no usage records, and Claude Code's 30-day transcript retention erased the only other trace. The lesson that frames the whole piece: **if measurement isn't part of the pipeline, it doesn't exist.**

The second-generation factory fixes this by persisting its own bill as it works: every stage attempt writes its tokens (input, output, cache read, cache write), cost, model, and failure category to a ledger. Using that ledger and the raw session logs (which turn out to be the ground truth), fxmartin prices a *story-build* — a user story run through its full delivery cycle: tests first, build, coverage gate, reviewer-agent review, merge, plus bugfixes and re-ask loops.

The headline number: the factory consumed **595.7 million tokens to ship 77 stories — 7.7 million tokens per delivered story, $837.53 total, or $10.88 per story** (median $9.56, range $3.02–$43.24). Story points barely predict cost, and the mean wall-clock is 2.5× the median because an overnight run twice hit the subscription plan's rate-limit window — a billing artifact, not an agent one.

The estimate-busting finding is about cache: **95.4% of all tokens are cache reads** — "a dark code factory is mostly a reading machine that occasionally types." Cache *writes* are only 3.3% of tokens but **31.2% of the bill**; cache traffic overall is 77% of cost, fresh input a rounding error at 1.6%. So cost optimization in an agentic pipeline is *cache management, not prompt shortening*.

Two honesty moves distinguish the piece. The "honest denominator" counts all rework, retries, bugfix loops, and crashed sessions at ~13% of tokens, against the narrow 5.0% marked FAILED. And the meter itself lied: the ledger missed a sixth of real consumption ($694.65 vs. $837.53) because a failed-envelope re-ask overwrote the original stage row, and crashed sessions never wrote back. "The measurement system needed auditing just like the code it measures" — so fxmartin filed the bug against his own factory and let its fix pipeline repair it.

On who actually pays: the marginal bill was zero, because fxmartin runs a $200/month Max 20x flat-fee subscription. Measured API-equivalent spend across three codebases was ~$1,088/month — a 5× asymmetry against list rates that reflects Anthropic's margin, not a subsidy. The subscription's real currency is **quota, not money**; the overnight run stalled on the 5-hour rate-limit window, so "time is the fence." The flat-fee window won't stay open forever, and "a factory that meters itself will notice the day the trade turns."

The meter changed the work in two ways. Model routing was switched off the whole time, so mechanical merges burned premium-model prices on Haiku-grade work (12.3% of all tokens) — 7.7M tokens/story is the *unoptimized* rate. And pointing the meter at himself, fxmartin found that writing the factory's specifications consumed ~190 million tokens, roughly 25 stories' worth (~$160): "the code is no longer the expensive artifact."
