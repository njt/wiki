# AI Cybersecurity After Mythos — The Jagged Frontier

Stanislav Fort (AISLE) systematically tests Anthropic's Mythos vulnerability-discovery claims against cheap, open-weights models and finds that detection capability is broadly accessible — eight out of eight models, including a 3.6B-parameter one, detect Mythos's flagship FreeBSD exploit. The "jagged frontier" means cybersecurity capability doesn't scale smoothly with model size, and small models sometimes outperform frontier ones (notably on OWASP false-positive discrimination). The moat is the scaffold, the pipeline, and the triage layer — not the model.

---

## Key Quotes

> "The moat in AI cybersecurity is the system, not the model."

Fort's thesis in one line. The same architectural insight as [[Smart Models Dumb Pipes]] and [[Harness Engineering]] — the deterministic surround matters more than the probabilistic core. This isn't "Mythos isn't good," it's "don't let the framing convince you that only Mythos can do this."

> "Small, cheap models outperformed large frontier ones" on OWASP false-positive discrimination.

The inverse scaling result is the most striking empirical finding. Sonnet 4.5 and Opus 4.5 both confidently traced a non-existent data flow ("Index 1: param → this is returned!") while a 3.6B open-weights model got it right. This isn't just "cheap models are good enough" — it's "expensive models sometimes hallucinate security conclusions." That's a qualitatively different and more urgent problem for anyone trusting a single frontier API for vulnerability scanning.

> "A thousand adequate detectives searching everywhere will find more bugs than one brilliant detective guessing where to look."

The coverage-over-brilliance tradeoff, stated plainly. Fort's argument is that cybersecurity is a search problem (echoing [[Cybersecurity Is Proof of Work Now]]), and search problems reward parallelism over per-unit intelligence. This is the scaling argument for open-weights models in security: you can afford to run them everywhere.

> "This is exactly why the scaffold and triage layer are essential."

After the April 9 update revealed that only one model was perfectly reliable on patched code (3/3 correctly calling it safe), while others hallucinated signed-integer bypasses that were impossible (the field is `u_int`, unsigned). Sensitivity was 100% across all models; specificity varied wildly. The scaffold — not the model — is what turns raw detection into trustworthy verdicts.

> "With actual tool access, the gap would likely narrow further."

On exploitation vs. detection. Fort concedes Mythos's multi-round constrained-delivery mechanism is genuinely novel engineering, but notes he only tested models with prompt access, not agentic infrastructure. This is a fair limitation — but it's also a motte-and-bailey: the whole point of Mythos is that it's an *integrated system*, not just a model. Testing the model without the system and concluding the model alone approaches the system's performance is informative but incomplete.

---

## Key Themes

#concept #security #model-evaluation #open-source #scaffold

**The jagged frontier in cybersecurity.** The concept that capability doesn't scale monotonically with model size. Some models are better at specific vulnerability classes, some at false-positive discrimination, some at exploit reasoning. No single model dominates. This has practical consequences: a multi-model ensemble with cheap, diverse models may outperform any single frontier model for detection coverage.

**Sensitivity vs. specificity.** Fort's April 9 update is the most important methodological contribution. 100% sensitivity (finding bugs in unpatched code) combined with poor specificity (falsely flagging patched code) is a worse failure mode than the reverse — it trains maintainers to ignore alerts. The scaffold's triage function (verifying whether a flagged vulnerability is real) is where the real engineering lives. This connects to [[Benchmark Exploitation]]'s insight that benchmarks measure what's easy to measure, not what matters.

**Coverage as the real metric.** Fort's "thousand adequate detectives" argument reframes AI security tooling from "which model is smartest" to "which pipeline has the widest search aperture." This has direct implications for [[Local and Open Source Inference]] — if coverage beats per-unit intelligence, the ability to deploy models on your own hardware at scale matters more than access to the most advanced API.

**The model-scaffold boundary.** Fort's tests isolate model capability (give it the function, ask it to reason) from system capability (give it the codebase, let it search, verify, and triage). The former is what most benchmarks measure. The latter is what actually matters in production. The gap between them is where [[Harness Engineering]]'s feedforward/feedback framework applies — and where AISLE's own production system (180+ CVEs across 30+ projects) demonstrates what a mature scaffold looks like.

---

## Critical Analysis

**What's strong:** The empirical approach is exactly what this conversation needed. Instead of "Anthropic says Mythos is special" vs. "open source is just as good" as tribal identity, Fort runs the actual tests and reports the actual results. The finding that 8/8 models detect the FreeBSD bug — and that one of them is 3.6B parameters at $0.11/M tokens — is a concrete, actionable data point for any team building AI security pipelines. The inverse scaling on OWASP false-positives is genuinely surprising and important.

The scaffold-over-model thesis is right, and it's right for reasons that extend beyond cybersecurity. [[Harness Engineering]] makes the same argument for coding agents; [[Smart Models Dumb Pipes]] makes it for AI system architecture generally. Fort's contribution is demonstrating it empirically in the security domain with direct head-to-head tests against the most prominent frontier security model announcement.

**What's weak:** Fort's framing as a "rebuttal" is somewhat artificial. He's not really rebutting Anthropic's claim that Mythos is good at finding vulnerabilities — he's rebutting the *implied exclusivity* of that claim ("this work depends on a frontier model"). But that implication is more in the PR framing than in the technical content. Mythos's genuinely novel contribution — the multi-round constrained exploit delivery for the OpenBSD bug — is the one piece Fort's cheap models couldn't replicate. That's worth more attention than he gives it.

The testing methodology, while rigorous, has a blind spot: Fort gave models the vulnerable function directly (described as "an upper bound" on autonomous performance). That's fair as a ceiling test, but the hard part of vulnerability discovery isn't reasoning about a function you're handed — it's finding that function in a 27-year-old codebase in the first place. AISLE's own production system handles this through broad-spectrum scanning (their "thousand detectives"), and Fort is arguing this is the model-appropriate approach. But he hasn't demonstrated that cheap models can do the *end-to-end* discovery pipeline without a frontier model somewhere in the loop.

**The framing problem is real.** Fort's main concern — that "requires a restricted frontier model" discourages adoption — is a legitimate worry. If organizations believe AI security requires Mythos-level capability and they can't access Mythos, they'll do nothing. Fort's message ("the models are ready, start building scaffolds") is the right one, even if his empirical case slightly overstates how much of Mythos's capability cheap models can replicate end-to-end.

**What this means for the wiki:** This piece is the most direct complement yet to [[Cybersecurity Is Proof of Work Now]]. Breunig argues security is a compute economics problem; Fort provides the empirical evidence that the compute doesn't need to be expensive. Together they make the case that AI cybersecurity is accessible *and* that its cost structure is adversarial (you must outspend attackers). The tension is productive: Fort says "cheap models work," Breunig says "but you still need to spend more than your adversary." Both can be true — the scaffold scales with compute, not the model.

The inverse-scaling finding deserves its own note in the [[Benchmark Exploitation]] context: a 3.6B model outperforming Opus 4.5 on a reasoning task is the kind of anomaly that should make everyone suspicious of single-model benchmark claims in security.

---

*Sources: [[summary/ai-cybersecurity-after-mythos-the-jagged-frontier]]*
*Source URL: https://aisle.com/blog/ai-cybersecurity-after-mythos-the-jagged-frontier*
*Author: Stanislav Fort, AISLE*
*Published: 2026-04-07 (updated 2026-04-09)*
*Fetched: 2026-05-15*
