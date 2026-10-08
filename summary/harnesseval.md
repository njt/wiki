---
url: https://dsifry.github.io/harnesseval/
title: "harnesseval — Find More Bugs with the Model You Already Use"
author: Dave Sifry
date_fetched: 2026-10-08
date_published: 2026
topics:
  - ai-code-review
  - guardrails-and-feedback-loops
---

Dave Sifry's open harnesseval study compares eight models on AI code review under two regimes: a single carefully-written review prompt, and free harnesses (Compound Engineering and metareview — the latter written by Sifry himself, a disclosed conflict of interest) at three effort levels. Six selected pull requests from two codebases, 147 verified bugs, one run per setup, all data and code public.

The headline: the harness matters more than the model. With the same model and effort level, a harness found more verified bugs in 39 of 42 comparisons — a median 1.6× gain, 13.5 percentage points of recall on average. The best harness setup found 88 bugs; the best single-prompt setup found 54. The price: roughly 10× the tokens, 5× the cost, 3.5× the review time, and more unsupported findings for a developer to check.

On cost, GLM-5.3 running metareview at low effort ($0.22/review, 72 bugs) matched Opus 5 with the same harness ($2.97/review, 74 bugs) at the same quality score — 1/13 the price, and the cheap model is open-weight. On effort, high beat medium in only four of 22 comparisons while costing more in 20 of 22; the study's advice is to default to low or medium and make high effort earn its bill on your own code. An interactive explorer lets teams compare all 66 setups and, crucially, run their own evals on their own workloads.
