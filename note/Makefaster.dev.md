# Makefaster.dev

Makefaster.dev is a landing page for an autonomous performance-optimization skill: an AI agent, installed with a single `npx` command, that loops forever over your website — researching issues, testing hypotheses in isolation, applying improvements, measuring, repeating — and claims average LCP reductions of 59% and TTI reductions of 49% across a public leaderboard of 199 sites.

---

## The argument in one paragraph

Makefaster.dev claims that web performance optimization has become fully automatable: an AI skill running an autonomous research loop can continuously discover, test, and implement performance improvements to a site without a human in the loop, and — because it is seeded with learnings from 200 prior "Fable max thinking runs" on the top 200 starred GitHub repos with frontends — it arrives at your site already knowing the moves. The falsifiable core is the number: that across 199 real sites on a public leaderboard, one autonomous run per site cuts cold-load LCP by 59% and TTI by 49% on average. If true, this is a strong data point that a narrow, well-instrumented domain with objective metrics (Core Web Vitals) is where autonomous agents deliver reliably; if the averages are cherry-picked or the leaderboard is self-selected, it is marketing dressed as evidence.

## Key quotes

> "Makefaster.dev is an AI skill that runs an autoresearch loop to continuously discover, test, and implement performance improvements."

The product thesis in one sentence: not a lighthouse audit, not a consultancy — a *loop*. The framing borrows directly from the agent-loop literature (research → experiment → implement → repeat) and sells persistence ("continuously") as the feature.

> "It uses learnings from 200 Fable max thinking runs performed on the top 200 starred github repos with frontents."

Note: quoted verbatim from the source, which reads "frontends" — the claim is that prior expensive reasoning runs on popular repositories were distilled into transferable knowledge. This is the most interesting and least substantiated idea on the page: performance patterns (bundle splitting, image handling, hydration cost) may well generalize across frontend codebases, but nothing on the page says what the "learnings" actually are or how they are injected.

> "Generates and tests hypothesis in a safe, isolated environment."

The typo ("hypothesis" singular) aside, this is the safety claim, and it is doing a lot of work in four words. Isolated how — a preview deploy, a sandboxed build, a shadow environment? And safe for whom: the production site, the users, the git history? The page never says.

> "Average Improvements — Averaged across the public site leaderboard — one run per site, the cold load where a site has both."

The methodology footnote, such as it is. One run per site, cold load only, averaged across a public board. This is the closest the page comes to honesty about its evidence — and it quietly concedes that the numbers are averages over an unspecified distribution, on the easiest-to-game measurement (first load), on sites whose owners chose to run a performance tool publicly.

> "Applies the best improvements automatically."

The step where the agent writes to your codebase. No mention of review, diff approval, or rollback. "Automatically" is the selling point and the risk in the same word.

## Critical analysis

The non-obvious thing here is the *seeding* claim. Most agent-loop products sell the loop; makefaster sells the prior — 200 expensive reasoning runs on popular repos, compressed into learnings that make run 201 cheaper and better. If that transfer is real, it is a small demonstration of an economics the wiki keeps circling: one-time deep reasoning amortized across many cheap autonomous runs. It is also the claim most in need of evidence and the one least capable of being checked from a landing page.

The weak points are everywhere the page is silent. Averages across 199 heterogeneous sites hide exactly what matters — the wiki's own [[Benchmarking AGENTS.md Changes]] finding that average improvement can mask task-specific regression applies directly here: a -59% mean is compatible with most sites gaining little and a few gaining enormously. Cold load is the most optimizable and least representative metric; SPA route transitions and warm navigations, where real users spend their time, are unmeasured. "Safe, isolated environment" is asserted, never described — there is no word about how an implemented improvement is verified as behavior-preserving, which is the entire hard problem in autonomous code modification. And the leaderboard is both evidence and funnel: sites that run a performance tool and publish results are precisely the sites with the most headroom, inflating the averages.

What is left out: cost (how much model spend per run?), what the agent actually edits (source, build config, CDN settings?), what happens when an improvement breaks something, and whether the loop ever stops or plateaus. "Always getting faster" is either a promise of unbounded improvement — implausible — or an always-on agent with standing write access to your site, which deserves a security and review conversation the page does not start.

## Related

- [[Performance Optimization Loop]] — Makefaster is essentially that page's loop (profile, hypothesize, change, measure) productized as an always-on autonomous skill; it strengthens the claim that performance work is loop-shaped, and adds public aggregate numbers the practice pages lack.
- [[Optimise Anything]] — That source argues for one general optimization API over any measurable artifact; makefaster is a narrow vertical instance (Core Web Vitals only), which nuances the thesis by suggesting the market is pulling toward domain-specific packaged loops rather than general ones.
- [[Agent Skills Library (dzhng)]] — dzhng's skills compose into a general development workflow; makefaster shows the opposite pole, a single-purpose skill running unattended as a service, complicating the idea that skills are primarily building blocks rather than standalone products.
- [[Benchmarking AGENTS.md Changes]] — That note warns that average improvement across tasks can mask specific regressions; makefaster's headline -59%/-49% averages over 199 self-selected sites invite exactly the same skepticism about means hiding distributions.

---
*Sources: [[raw/makefaster-dev]], [[summary/makefaster-dev]]*
