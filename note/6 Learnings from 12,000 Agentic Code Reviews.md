# 6 Learnings from 12,000 Agentic Code Reviews

Watson Labs shares six months of metrics from "The Mandible" — a software factory built for stealth Swamp customer Gymwasp, where no human reads any code. Seven isolated reviewer lanes gate every merge worst-of-seven across 372 issues, and the numbers say the review loop converges fast then oscillates: most value lands in round one, round 4 is the elbow, and 29% of issues ship on warn rather than chasing a clean pass. The real contribution isn't the factory — it's the measurement discipline: survival analysis over naive averages, per-lane drift gauges, and the CI/CD retune loop applied to review itself.

---

## Key Quotes

> "**65% of issues are merge-ready after a single agentic review round.** Most of the value lands immediately. **After round 4, extra rounds introduce as much uncertainty as they're trying to fix.**"

The two halves of the convergence story. The front-loaded 65% is the encouraging half; the oscillation tail is the one practitioners need more. "The loop oscillates rather than converges" is the most actionable finding in the piece — it tells you *when to stop*, which is the question every review loop eventually faces. The conditional pass rate halves after round 2 and flatlines near 87% cumulative: additional rounds stop buying quality and start buying churn.

> "**29% of issues never reached a clean pass.** They shipped on warn. This is their measure of risk tolerance."

Ship-on-warn as an explicit, measured dial rather than a dirty secret. The author is blunt about the alternative: "Chasing 100% on a complex PR given the non-determinism of LLM-based reviews is not a productive use of compute. Ship at merge-ready, batch the follow-ups." That's a risk posture you have to decide, not stumble into — and the warns are logged, so the follow-ups at least exist as debt entries.

> "**The naive average understates cost by 44%.** If you only count the issues that clean passed, you get 3.31 rounds. The honest number, when including issues that shipped with warnings, is 5.99."

The best statistical point in the article. Averaging only over clean passes is survivorship bias applied to your own pipeline — the hard issues vanish from the denominator exactly because they're hard. "The censored observations are data, not noise" is the survival-analysis framing, and it's rare to see an agent-pipeline write-up this honest about its own denominator.

> "Each reviewer gets a narrow brief *and* an explicit out-of-lane exclusion list. Without exclusion lists, you get the same finding seven times."

Negation as the signal-to-noise lever. This independently reproduces Cloudflare's negation-first prompt engineering — "telling an LLM what NOT to do is where the actual prompt engineering value resides" — as a structural requirement of parallel review: seven agents with overlapping briefs don't give you seven opinions, they give you one opinion seven times.

> "**1,801 adversarial review rounds: 1,463 on code, 338 on plans.** The agent is far more likely to write the code wrong than write the plan wrong."

The first quantification I've seen of the spec-first intuition. Plans are reviewed by the same adversarial machinery as code, and they fail at roughly a fifth the rate. It's evidence that the cheap place to catch errors is before code exists — with the obvious caveat that plans are smaller artifacts, so they have less surface to get wrong.

## Key Themes

#pattern #concept #tool #person

- **#pattern — Worst-of-seven ensemble review.** Seven lanes (test coverage, clean code, frontend, DDD, security, accessibility, observability), each on its own model instance, blind to the others, returning pass/warn/fail; the round's verdict is the worst of the seven. A mechanical, pessimistic version of the ensemble review proposed in [[The End of Code Review]] — and the opposite synthesis choice from Cloudflare's coordinator-judge in [[Orchestrating AI Code Review at Scale]].

- **#concept — The review elbow.** Review rounds are not a monotonic quality accumulator. Value lands in round one, converges through three, and after round four the loop oscillates — more rounds add as much uncertainty as they fix. The fix at the elbow is editorial (tighten briefs, sharpen exclusion lists, concretise the warn/fail threshold), not computational.

- **#concept — Censored observations are data.** Naive averages over clean passes understate cost by 44% (3.31 vs 5.99 rounds). Survival analysis, not averages, is the honest way to measure a loop where some issues never terminate cleanly.

- **#pattern — Drift gauges on the review pipeline.** Four failure signatures with four fixes: pass rate stagnating → retune briefs; one lane dominating fails → brief too aggressive or genuine signal; infra flakes eroding trust (3.7% of verdicts lost to agent crashes, 8.1% of rounds with an unassessed lane) → "fix the brief, don't kill the lane"; the elbow migrating right → stale briefs and exclusion lists. Instrument → measure → find the elbow → retune → measure again.

- **#concept — CI/CD discipline, new target.** The author's explicit thesis: thousands of managed CI/CD pipelines transfer directly — a deterministic pipeline with measurable, instrumentable stages. "The old discipline was making builds fast and tests reliable. The new discipline is making review accurate and review loops short."

- **#tool — Swamp as the deterministic harness.** Every number came "out of swamp's data plane as a byproduct of the work" — the harness controls what each reviewer sees and tracks every verdict, which is why per-issue review cost stayed flat (median 4 rounds) while volume tripled. See [[Swamp Club]] and [[The Lifecycle of a Swamp Issue]].

- **#person — James Owens** shared the Gymwasp data; the author blogs at Watson Labs and runs "the factory we run internally for Swamp itself."

## Critical Analysis

**Unpack the title before citing it.** "12,000 agentic code reviews" is ~1,801 adversarial review rounds across 372 issues, with seven lane-verdicts per round — a "review" here is one lane's verdict on one round, not a human-equivalent PR review. The arithmetic is consistent (7 lanes × ~1,700 rounds ≈ 12,000 verdicts), but the unit flatters the volume; what you actually have is 372 issues' worth of evidence.

**The numbers measure the loop, not shipped quality.** "65% merge-ready after one round" means the seven lanes passed it — the same system that generates the code grades it, and the briefs are the whole quality bar. There's no external ground truth in the post: no post-merge defect escape rate, no human spot-check audit. The 29% ship-on-warn figure is admirably honest, but "batch the follow-ups" presumes the warns actually get paid down, which the data doesn't show. This is the measurement circularity that [[Agentic Code Review]] and [[Conquering Entropy — Cultivating Trust]] both worry about, wearing a metrics costume.

**Vendor provenance, and an unresolved identity question.** The piece ends as a Swamp ad ("Check it out at swamp-club.com"). The author sells the harness the numbers vindicate — flat per-issue cost at 3× volume is a harness feature as much as a finding. And note the domain discrepancy: this wiki's [[Swamp Club]] page records the framework at swamp.club (System Initiative), while this article points at swamp-club.com. Everything in the piece — deterministic automation, the data plane, adversarial plan review, the internal factory "for Swamp itself" — reads as the same Swamp ecosystem Paul Stack describes dogfooding in [[The Lifecycle of a Swamp Issue]], but the article doesn't resolve it, so treat the identity as very likely rather than certain.

**The methodology is the transferable part, and the author knows it.** Footnote iii says so plainly: every factory is tailored; results will vary. What travels isn't the 65% or the round-4 elbow — it's the instruments: convergence curves instead of throughput snapshots, censoring-aware cost accounting, per-lane fail breakdowns, flake rates, and elbow position as a drift alarm. Most teams, as the article notes, measure review by lead time or issue count, which tells you nothing about whether the loop converges.

**The oscillation finding is the piece that complicates loop engineering.** The emerging loop-engineering canon — [[108 PRs in Eight Days — Accidentally Discovering Loop Engineering]], [[Poor Man's Loop Engineering]], [[Feedback Loop is All You Need]] — is largely enthusiastic about iteration as such. This data says iteration has a sharp elbow, past which the loop spends compute to move defects around. Self-terminating loops and batched follow-ups stop being stylistic choices and become what the convergence data recommends.

## Cross-Links

- Strengthens [[Orchestrating AI Code Review at Scale]] by independently reproducing its negation-first finding as a structural requirement (exclusion lists), and complicates its architecture: where Cloudflare synthesizes seven reviewers through a coordinator-judge into one comment, The Mandible has no judge — the worst of seven blocks the merge. It also adds the dimension Cloudflare's per-review snapshots lack: per-issue convergence across rounds.

- Strengthens [[The Lifecycle of a Swamp Issue]] with the quantitative counterpart to its adversarial-plan-review phase: 338 adversarial rounds on plans against 1,463 on code — the same Swamp ecosystem, now with six months of factory data behind the lifecycle Paul Stack described.

- Strengthens [[Cloud Software Factories]] by supplying what Zach Lloyd's blueprint demands and mostly lacks: a running instance with honest per-issue metrics — rounds-to-merge-ready, censoring-aware cost, flat per-issue review cost at tripled volume — captured as a byproduct rather than archaeology.

- Strengthens [[The End of Code Review]] by operationalising Monperrus's agent-in-the-loop verification with production data: no human reads code, humans at Plan and Review assert problem-understanding and make product calls, and ship-on-warn is the explicit risk-tolerance dial his "review as engineering check" framing implies.

---
*Sources: [[raw/6-learnings-from-12000-agentic-code-reviews]], [[summary/6-learnings-from-12000-agentic-code-reviews]]*
*Last updated: 2026-09-13*
