---
url: https://danluu.com/ai-coding/
title: "Agentic test processes, LLM benchmarks, and other notes on agentic coding from Galapagos Island"
author: Dan Luu
date_fetched: 2026-07-29
date_published: 2026-07
---

# Agentic test processes, LLM benchmarks, and other notes on agentic coding from Galapagos Island

Dan Luu

The piece opens with an anecdote about asking an earlier GPT model (possibly 5.0 or 5.1) to find the source of a UI interaction bug. The agent made up a fabricated reproduction complete with a video showing the "bug" — but the video came from an artificial browser environment, not the real one. Luu's reaction: "I immediately thought to myself, 'how can I get more of this?'"

## Testing background

Luu argues that LLMs are "highly leveraged when it comes to testing" — making it easier than ever to reach a given quality bar — yet "software seems to be lower quality than ever." He describes a pipeline from support ticket to pull request that has yielded no known false positives.

He references a "software factories" workflow he's comfortable with, noting it achieves "much higher quality than any review-reliant workflow I've seen or even heard of."

The article then draws heavily on the author's experience at Centaur, a hardware company he worked at for his first decade in the industry. Key practices there included:

1. Dedicated QA/test engineers with career parity to developers
2. No code review by default
3. Virtually no hand-written tests
4. Constant randomized/property-based testing (called just "tests" — hand-written ones were "hand tests")
5. A large regression suite (3 months wall-clock to execute on a compute farm)
6. No unit tests

At the time Luu left (2013), they had about 1,000 machines generating and running tests continuously for roughly 20 logic designers and 20 test engineers. About 80% of machines generated new tests; 20% ran regressions. Pre-commit tests took ~10 minutes on overclocked machines. New failures were triaged by dedicated engineers.

The author notes that "unless you count culture as a separate item," item (1) was the biggest difference from typical software companies. Testing as a skill improves with dedicated practice, much like distributed systems or UX.

On code review: "We didn't review code by default because we trusted our test practices enough that review didn't, in general, add much reliability." They shipped fewer than one significant user-visible bug per year. Luu expresses comfort shipping code without human review, and pushes back on risk concerns from companies with much higher bug rates.

On testing methodology: randomized test generation is far more efficient than hand-written tests. Keeping regression tests forever leads to large suites, but running the same tests repeatedly in CI is inefficient compared to running many different tests. Unit testing — from an efficiency standpoint — "does pretty poorly."

Luu addresses the common objection that CPU design is fundamentally different from software: "When I first switched from CPU design to software I thought that might be true, but I've since tried this testing methodology with every kind of Y that someone has mentioned this can't work for and it's worked for every single one."

The effort ratio was about 1:1 test engineers to developers, plus ~10% time in "freeze" states. A rough estimate: 55% effort on testing, 45% on development. Luu argues the effort ratio isn't what makes the difference — methodology matters more.

On LLM-vs-fuzzing for bug finding: "fuzzing generally wins on latency to find a bug, and it dominates on finding more bugs and having a lower false positive rate."

## Some details on testing

Luu says "LLMs seem pretty bad at testing" when left to their own devices but can be steered effectively. People who find LLM-generated tests "great" are typically those who did essentially no testing before.

As of June 2026, directing LLMs to generate fuzzers — "for most projects, this will turn up real and often serious bugs within minutes." However, the coverage is "curiously bad" and misses basic things. LLM-generated fuzzers "don't do a good job of 'thinking about' how inputs should be varied to elicit bugs."

For a "software factories" workflow shipping hundreds or thousands of PRs per day, "everything that's not constrained from degrading will rapidly degrade." The system needs feedback loops — whether human input, staged rollouts, or monitoring metrics/logs/traces/support tickets. His support-ticket-to-PR pipeline tries to add test coverage that will re-find bugs if they regress.

The author asks why LLMs are bad at writing tests and is told it relates to RL environments and a thin market for selling them. He invites readers who can connect him with buyers at labs to reach out.

On detecting false positives: even access to models better than anything publicly available "won't save you." He recounts Dennis Snell receiving "AI slop" forwarded by Anthropic from their unreleased Mythos model — garbage with no reasonable false positive rejection. Meanwhile, Luu himself, using a supposedly less capable model, had "no problem generating an endless stream of bugs... with no known false positives." The lesson: "having a reasonable setup around the model is a least as important as having the latest and greatest model."

Semi-generalizable techniques include: having independent agents repeatedly check alleged bug reproductions; using different "personas" for reviews (plus "contrarian" personas); producing artifacts like videos; having agents review those artifacts; and getting "independent perspectives." He notes "Pretty much everything I've tried to reduce false positive rate has worked."

## Caveman mode

Luu investigates "caveman mode," which purportedly reduces token usage and speeds up prompt resolution (claimed: 75% reduction, 2x fewer tokens, 3x speed increase). The creator later said on Hacker News that it's a joke, but most online recommendations are positive. At work, when Luu asked if anyone had done a comparison, someone linked "an LLM-generated SEO spam article with numerous errors."

So Luu spent about 15 seconds per benchmark generating comparisons. Using GPT-5.5 xhigh on a wasm code optimization task, initial results looked promising for caveman. After 50 runs, the average showed a 1.03 vs 1.01 speedup favoring caveman, with lower cost ($17.97 vs $24.21) and faster wall-clock (13m46s vs 16m52s).

P-values (per a GPT-produced statistical script): for speedup p=0.1, cost p=0.005, wall-clock p=0.001. Bayesian results showed P(caveman better) at 0.958, 0.999, and 1.000 respectively.

But on two other benchmarks — another wasm optimization and a board game AI (Lost Cities, 10ms per move) — results flipped. For Optimization 2: P(caveman better) = 0.17 for speedup (worse), though cost and time savings held. For Game AI: P(caveman better) = 0.04 for quality (worse), with mixed cost/time results.

Testing across more models (GPT-5.4 mini, GPT-5.4, GPT-5.5) at every effort level revealed: "there's enough variance between conditions... that it's clear that we'd have to run a lot more conditions to get a clear picture." The overall difference "averages out to be small enough that it doesn't seem worth using caveman mode."

## LLM variance

When GPT-5.5 was released, people said contradictory things: that 5.4 is better at staying on task; that 5.5 is so much better it's cheaper overall; that 5.5 can run at lower effort levels; that 5.5 "just works" while 5.4 often fails. All of these statements find support in Luu's benchmarks — the results depend heavily on the specific task.

"With just these three evals, you can find support for every statement I saw people making about GPT-5.5 on release because all of the statements are sometimes true."

This is why Luu finds single-number summary benchmarks "basically meaningless." If the benchmark set were slightly different, results would flip. On DeepSWE-style benchmarks, most tasks get either 4/4 or 0/4 with SOTA models, and "a small subset of tasks that actually determine the relative scores." Changing a few tasks out of ~100 can flip which model appears to lead.

This reminds Luu of Miguel Indurain — a household name in cycling due to arbitrary factors about which race (Tour de France) became famous and which stage types happened to dominate during his era. "You tweak the benchmark a bit and Miguel Indurain goes from being a once household name to an all-time great time trialist that pretty much nobody has heard of unless they follow cycling."

On whether these benchmarks matter: despite GPT-5.5 handily beating Opus 4.x models on most benchmarks, "Anthropic's business grew much faster than OpenAI's during the time period." Anthropic's revenue trajectory "is incompatible with these benchmarks being major determinants of user choice."

Variance is also large *within* a single task for the same model. For GPT-5.5 xhigh on Optimization 1, one standard deviation between runs is 7.5% — larger than the difference between the best and worst tested GPT variant.

The article continues beyond this point (content was truncated in the fetch).

---

Key themes: Luu argues that the methodology around AI models matters far more than which specific model you use; that randomized/fuzz testing is underutilized in software and extraordinarily effective; that most benchmark comparisons are noisy to the point of meaninglessness; and that the hardware-testing practices he used at Centaur are surprisingly transferable to modern AI-assisted software development.
