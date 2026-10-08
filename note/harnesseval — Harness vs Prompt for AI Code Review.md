# harnesseval — Harness vs Prompt for AI Code Review

Dave Sifry's open study of AI code review finds that the review *workflow* — a harness with multiple passes, tools, and specialist reviewers — beats the model choice: across 42 same-model, same-effort comparisons, harnesses found more verified bugs in 39 of them (median 1.6×), and an open-weight GLM-5.3 running a $0.22 harness matched Opus 5 at $2.97. High effort rarely pays. One run per setup and six selected PRs mean the numbers are directional, but the data and code are public.

---

## What the study is

Eight models × a carefully-written single review prompt × two free harnesses (Compound Engineering and metareview) × three effort levels, over six selected pull requests from two codebases with 147 verified bugs. One run per setup. Everything — data, code, charts, an interactive 66-setup explorer — is public. Full disclosure: Sifry wrote metareview, one of the two harnesses under test. That disclosure is stated in every section of the page, which is more than most evals manage.

## Key quotes

> "Your AI code reviewer may not need a smarter model. It may need a better workflow."

The thesis, and the framing this wiki keeps circling: the harness is where the leverage is. This is the review-side counterpart to the loop/harness/factory stack in [[Software Factories, Light and Dark]] — measured, with bugs verified by humans, rather than asserted.

> "The gain has a price: median token use was about 10× higher, cost 5× higher, and review time 3.5× longer. There were also more unsupported findings to check. A higher bug count does not automatically mean less work for a developer."

The honest sentence most harness marketing omits. Recall is not the objective function; developer minutes spent triaging unsupported findings are a real cost the quality score only partially penalises. This is the same false-alarm economics Cloudflare had to engineer around at production scale in [[Orchestrating AI Code Review at Scale]] — their "tell the LLM what NOT to do" prompt discipline is the single-prompt answer to the same problem harnesses solve with more passes.

> "GLM-5.3: $0.22 per review, 72 bugs found across the test set. Opus 5: $2.97 per review, 74 bugs found."

The cost finding is the one with business consequences: an open-weight model plus a free harness at low effort matches the frontier flagship's score at 1/13 the price. It strengthens the open-weight value case made in [[Local Qwen Is Not a Worse Opus]] with review-task data instead of vibes — though Sifry is careful to say it's a reason to choose an open-weight setup *for value*, not a claim that all models perform equally.

> "In 17 of 22 AI code review comparisons, the study could not establish a quality difference between high and medium effort. High cost more in 20 of 22."

"Think harder is easy advice when someone else pays the bill." Effort settings are a pricing dial, and the study's evidence says most teams should turn it down and prove otherwise locally. The epistemic discipline matters too: "grey does not mean no effect," it means the sample can't tell.

## Key themes

#concept (harness over model) #tool (metareview, Compound Engineering) #pattern (evaluate on your own code) #comparison (66 setups, price vs quality frontier)

## Opinionated take

This is what a honest vendor eval looks like: disclosed conflict of interest, public data, explicit uncertainty ("small score differences are not reliable rankings"), and a refusal to oversell — the study never claims harnesses save developer time, only that they find more verified bugs. The weaknesses are real and stated: six selected PRs, one run per setup, a quality score designed by the study's author, and no measurement of triage time, which is precisely where harness findings get expensive. The three harness losses were all one model on one harness (Sonnet 5 + Compound Engineering), which hints at harness-model interaction effects the single-run design can't resolve. Still, the direction of the result — workflow over weights — is consistent with everything else in this wiki, and this is the first source here that puts review-specific numbers on it.

The uncomfortable corollary: if a harness at low effort finds 2.1× the bugs, then most teams running single-prompt reviewers are not limited by their model budget — they're limited by never having run an eval. The study's actual product is the habit, not the leaderboard.

## Related pages

- Strengthens [[Orchestrating AI Code Review at Scale]]: Cloudflare's seven-specialist-agents-plus-judge production system is an instance of exactly what this study measures — harness beats prompt — here quantified across 42 controlled comparisons.
- Nuances [[Local Qwen Is Not a Worse Opus]]: the open-weight-beats-frontier-on-value claim gets its first review-task measurement, with the important caveat that the match required the harness, not the model alone.
- Complicates [[Goodhart's Law and AI Benchmarks]]: the quality score is Sifry's own construction rewarding verified bugs and penalising unsupported findings — a reasonable metric that is still the eval-author's metric, and overlapping uncertainty ranges at the top of the leaderboard mean the ranking is soft.
- Extends [[Compound Engineering Plugin]]: one of the two harnesses tested is Every's plugin, here treated as an evaluation subject with measured recall and cost rather than a design essay.

---
*Sources: [[raw/harnesseval]], [[summary/harnesseval]]*
*Last updated: 2026-10-08*
