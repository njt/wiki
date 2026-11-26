# Perplexity as a Safety Pacing Metric (OpenAI)

OpenAI proposes cross-entropy loss — perplexity — as the shared industry metric both for comparing pretraining progress and for setting common capability limits, and puts its own internal loss numbers on the table to make the proposal credible. The bet: one cheap, stable, ungameable-ish number can anchor safety pacing where task benchmarks keep saturating.

---

The core claim is a compression worldview: modern AI is less a diverse toolbox of techniques than the *harnessing* of intelligence that falls out of training a compressor on a very large corpus. Everything downstream — prompting, finetuning, RL — is elicitation of a substrate whose progress is best tracked by held-out prediction loss. OpenAI publishes its curve from GPT-3.5 onward: ~0.99 log-perplexity bought fluent conversation, ~0.91 bought productive chain-of-thought, below 0.8 bought "transformative" capability in cybersecurity and mathematics, and the next window is projected at 0.68–0.7.

The policy ask is the unusual part: frontier labs should agree on transparency standards, joint red-teaming norms, and **common limits** — perplexity thresholds (plus other measurements) that developers commit not to cross until a concrete monitoring milestone exists, such as a successor to chain-of-thought monitors. It is an explicit pacing proposal, not just a measurement proposal.

#concept #pattern

---

## Key quotes

> "rather than designing something from scratch, they are mostly *harnessing* the intelligence coming from a much simpler and less interpretable process: training a compressor"

The framing that makes the whole argument work. If capability is a monotone function of one scalar, then one scalar can pace the industry. Note the honest parenthetical: the process is "less interpretable" — the substrate is exactly the thing we *can't* audit, which is why they want a proxy gate.

> "it was not at all clear *a priori* that these capabilities would appear once those precise perplexity thresholds are met"

The strongest sentence in the post, and it quietly undercuts the policy proposal. If threshold-crossings are ex-post discoveries, then a pre-agreed limit at 0.7 is a limit drawn on a map with blank regions inside it — you only find out what lives at 0.69 by going there.

> "seeing the resolutions to questions we've long thought about coming out of the model one after another in a single Slack thread felt deeply surreal"

The math-model anecdote (~0.76 log-perplexity producing groundbreaking results) is doing the accelerationist work: if one domain avalanched, pretraining research could too, "as soon as 2027". This is an extrapolation from n=1 and is the post's weakest link — but it's also the stated reason transparency can't wait.

> "chain-of-thought reasoning does not reduce perplexity achieved by the model, even though it leads to a drastic increase in capabilities"

The post's own list of caveats concedes the metric's biggest hole: the current source of most frontier capability gains is invisible to perplexity. A "first limit on development" measured in loss constrains the pretraining axis only.

---

## Critical take

This is a genuinely unusual document: a frontier lab publishing its internal loss curve and asking rivals to co-sign a ceiling on itself. Treat it with two kinds of suspicion.

**The Goodhart problem.** Once perplexity becomes the number that gates releases, it becomes the number labs optimise. Unlike task benchmarks, held-out next-token loss is hard to game *per se*, but dataset selection is not — footnote 4 admits OpenAI reports loss "on our internal codebase because I wanted to share the numbers we rely on most today", i.e. numbers favourable to its own story. Cross-lab comparison is the acknowledged weak point, and the proposal leans on a future "transferable" methodology that doesn't exist yet. See [[Goodhart's Law and AI Benchmarks]] for the structural argument that ungameable metrics don't survive becoming targets.

**The floor problem is a feature and a bug.** The post argues that as loss approaches the data's inherent entropy, small numeric gains mean large intelligence gains — which means a limit at 0.68 constrains far more capability than a limit at 0.9 did. That's presented as a reason the metric stays informative; it's also a reason the limits are not comparable across their own lifetime, and a reason a lab sitting just above the line has every incentive to argue the floor is closer than it looks.

**What it gets right.** Task benchmarks saturated around GPT-4 and this is true; a durable pretraining-side yardstick is a real gap. And publishing the curve is a costly signal — competitors get a progress picture for free — which buys the proposal more credibility than a pure op-ed would have. The honest thing to compare it against is [[On the Extinction Risk from Artificial Intelligence (RAND)]], which rejects probability forecasts under Knightian uncertainty: this post is essentially an attempt to convert Knightian uncertainty about emergent capability into a measurable, paceable quantity. Whether that's wisdom or metric-washing depends entirely on whether the limits are set by someone other than the labs crossing them.

---

## Related pages

- Strengthens the measurement-skepticism case in [[Goodhart's Law and AI Benchmarks]] — OpenAI proposes a metric precisely because public benchmarks got gamed, yet the proposal re-creates the same target-on-your-back dynamic at a higher level.
- Complicates [[On the Extinction Risk from Artificial Intelligence (RAND)]] — where RAND refuses to forecast extinction risk numerically, this post tries to pace the same frontier with a single scalar.
- Nuances the pretraining story in [[Recent Developments in LLM Architectures]] — architecture work optimises inference cost, and the post's own caveat says such gains don't show up in loss at all.
- Extends [[Harness Engineering (OpenAI)]] — same lab, complementary worldview: capability emerges from the compressor, but what you can *use* is determined by the elicitation harness built on top.

---
*Sources: [[raw/perplexity-and-intelligence]], [[summary/perplexity-and-intelligence]]*
*Last updated: 2026-11-26*
