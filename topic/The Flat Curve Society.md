# The Flat Curve Society

Steve Yegge's long-form argument that the AI intelligence curve is flattening — not because progress stops, but because the most dangerous models will be locked down like nuclear weapons, and because every human has a "discernment horizon" past which smarter models look identical. The essay is important as a synthesis of 2026's converging threads: Fable spooking governments, AI literacy emerging as the key org challenge, token efficiency becoming the new meta-skill, and SaaS roaring back when rebuild-everything economics hit the wall.

---

## Key Quotes

> "most of you aren't going to see it progress anymore"

The thesis in eight words. Yegge isn't saying progress stops — he's saying access stops. The curve continues behind velvet ropes. This makes him neither a doomer (the tech keeps advancing) nor a utopian (you won't get to use it). It's a distribution-of-power argument, not a capability argument.

> "You can't hand out an intelligence engine that nobody can supervise."

The cleanest statement of why superhuman AI gets locked down. Not because it's evil, not because it's a weapon — because it's *unverifiable*. If you can't check the work, you can't be responsible for the output. Safety people and enterprise buyers converge on the same conclusion for different reasons.

> "A plateau lets us set up a camp and start building."

The optimistic reframe that makes the essay more than a lament. If models can't get meaningfully smarter for most users, the engineering challenge shifts from "keep up with the next release" to "master the available tools." Stability as a precondition for craftsmanship. This echoes the scenius argument from [[Vibe Coding and the Maker Movement]] — a protected space where taste can develop — but at the level of the entire industry.

> "AI Literacy does not come for free. The only thing you get for free is AI Anxiety."

The best one-liner in the piece. Names the default state (anxiety) and the required investment (literacy) in a single sentence. This is the cultural challenge Yegge spends the back half of the essay on.

> "Token spend signals literacy on the way up, then flips to measuring token waste."

The bell-curve insight applied to AI usage. Beginners should be "absolute token pigs" — that's how they learn. Masters become token-efficient. The problem is organizations that treat tokens as cost-to-minimize from day one, starving the learning curve.

---

## Key Themes

- **#concept Discernment Horizon** — The two-ceiling model: the *demand horizon* (hardest problem you bring) and the *discernment horizon proper* (hardest answer you can judge). Past the second ceiling, models feel identical because you can't tell which is right. Everyone has one, including Dario. This is a genuinely novel framing that cuts through the "is it smarter?" debates.

- **#concept Nuclear Chokepoint** — Yegge's model for AI regulation: governments won't ban research, they'll lock the supply chain (compute, energy, chips) the way they locked uranium enrichment. This makes open-source models permanently trailing, not because they're worse engineering but because they can't access the inputs.

- **#pattern Token Literacy Curve** — Token spend as an inverted-U metric: rising spend signals growing literacy; falling spend signals mastery. The Netflix study provides hard data: 0M/4M/12-15M cohorts, 5-hour jump times, 96% retention. This is the most actionable framework in the essay for engineering leaders.

- **#concept SaaS Resurgence** — The buy-vs-build pendulum swinging back. Models capable of rebuilding SaaS exist, but access and cost make it impractical. Companies that blew yearly AI budgets in months are returning to SaaS. This contradicts [[The Road Runner Economy]]'s "one-shat" thesis — or at least pushes its timeline out.

- **#person Steve Yegge** — Veteran engineer (Amazon, Google, Grab). Known for long-form, opinionated essays. Previously wrote the canonical "Platform Rant" at Google. This essay reads as a return to form after years of cheerleading AI acceleration.

- **#person Ezra Savard (Netflix)** — Ran the AI literacy study Yegge cites. His training data — teams of 5-10 with their manager, real work, instructor facilitation, two 5-hour courses — is the benchmark for organizational AI literacy programs.

---

## Critical Analysis

**The nuclear metaphor is strong but overconfident.** Yegge assumes the supply chain is controllable the way uranium is — few sources, detectable, hard to hide. But compute is more distributed than enriched uranium. A GPU fab takes billions, but inference hardware is getting cheaper and more portable. The chokepoint holds for *training* frontier models; it may not hold for *running* them. If model distillation or peer-to-peer distributed training works at scale, the analogy breaks.

**The dual discernment horizon is the essay's most valuable contribution.** It explains a lot of confused discourse — why some people say Fable is revolutionary and others say it's the same as Opus — without needing either side to be wrong. The demand horizon particularly is underappreciated: if your hardest problem is "refactor this 200-line function," you literally cannot see the difference between models past a certain point because your problems don't stress either one.

**The SaaS section is the weakest.** Yegge argues that token costs make rebuilding SaaS prohibitive, but token costs are falling exponentially while software licensing costs rise. The crossover point moves every quarter. His "blowing yearly budgets in months" anecdote has truth to it, but it describes companies using AI *inefficiently* — exactly the problem the back half of the essay solves. A token-literate organization might find rebuilding cheaper than Yegge assumes.

**The Netflix study is the most actionable section and it's buried.** The specificity — 5 hours, 5-10 people, with manager, real work, instructor-facilitated — is a training playbook. The 96% retention figure is striking. Yegge's framing around token spend cohorts (0M/4M/12-15M) gives orgs a measurement framework. This deserves to be separated out as its own finding rather than a supporting argument.

**The essay dodges the "who controls the plateau" question.** If superhuman models are locked down to a select few, who are they? Yegge gestures at governments and frontier labs but doesn't explore what it means for a handful of institutions to have intelligence no one else can verify. The geo-strategic implications — one country with unverifiable AI, another without — are gestured at but not examined.

**Compare to** [[The Road Runner Economy]]: Raford argues AI is making software trivially replicable; Yegge argues the best models to do that replicating won't be available. Both can be right depending on timeframe — Raford describes what happens *if* access continues; Yegge describes why it won't. The tension between these two essays is the central strategic question of AI in 2026.

**Compare to** [[AI Killing B2B SaaS]]: That essay argues SaaS must become platforms to survive; Yegge argues SaaS is already rebounding because AI can't replace it cost-effectively. Yegge is more bullish on SaaS but less specific about what SaaS companies should actually do.

**Compare to** [[2026 Global Intelligence Crisis]]: Citadel's macro rebuttal to AI doomerism hits similar notes — S-curves, compute as boundary — but from a financial perspective rather than an engineering one. Yegge's "plateau" is Citadel's "S-curve flattening," reached by different reasoning.

**Compare to** [[Muse Spark and the Rough Edges Admission]]: Muse Spark is the thing Yegge is worried about — a superintelligence model released to billions. The "rough edges" line validates Yegge's argument that we're not ready for what's coming.

---

*Sources: [[summary/the-flat-curve-society]]*
*Last updated: 2026-06-22*
