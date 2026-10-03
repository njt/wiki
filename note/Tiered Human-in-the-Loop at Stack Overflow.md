# Tiered Human-in-the-Loop at Stack Overflow

Stack Overflow's engineering team tells the story of discovering that their human-in-the-loop framework — adopted for reliability, ethics, and compliance — had become the very bottleneck it was meant to prevent, and of rebuilding it as a three-tier system where a millisecond-classifying router decides which decisions a human ever sees. The result is one of the most concrete public write-ups of tiered review architecture: automated validation for 85% of traffic, asynchronous expert review for 12%, and 30–60 second real-time oversight for 3%.

---

## What it argues

HITL is an architectural choice with trade-offs, not a virtue. Applied blindly — a human checkpoint on every output — it introduces delay, cost, and reviewer fatigue without buying proportionate safety. The alternative is the Pareto principle applied to human attention: automate the high-confidence bulk, spend scarce human judgment on the low-confidence, high-risk, novel tail.

The engineering meat is the **Prediction Router**: a small Go model classifying each decision into one of three tiers in under 1 ms at 94% accuracy, trained on labels from previously validated human decisions. Its training objective is telling — maximise precision on Tier 1 (auto-resolve) and Tier 3 (expert review) even at the cost of Tier 2 spillover, so the fully-automated path is trustworthy and the critical path is rarely misrouted.

Three tiers, three latencies:

- **Tier 1 (85%):** rule-based validators, shadow models, anomaly checks — <3 ms added latency, zero human delay, gated by calibrated confidence (temperature scaling, isotonic regression).
- **Tier 2 (~12%):** asynchronous expert review in 4–6 hours, with active learning selecting the samples where human feedback teaches the model most.
- **Tier 3 (~3%):** real-time human oversight in a 30–60 second window with a conservative default if the reviewer doesn't answer.

Around the tiers: a Kafka review queue with "forced diversity" to prevent cherry-picking, a React/GraphQL review interface that pre-computes context (cutting review time from 6 minutes to 90 seconds), and a four-horizon feedback loop — Flink streaming for seconds-to-minutes situational awareness, nightly router retraining against routing drift, weekly model retraining plus reviewer-consistency audits, quarterly ROI analysis.

## Key quotes

> "60% of our time went to low-impact tasks, such as routine label checks, while high-value activities, like identifying new patterns, received only 15%."

The audit-before-optimising move. Most HITL budget is spent on work a sampler could do; the fix isn't more reviewers, it's measuring where human judgment actually earns its keep.

> "Reviewers did not take long to decide; they spent 5–10 minutes gathering context like training distributions or shadow predictions."

The bottleneck was never deliberation — it was context retrieval. Pre-computing the review dashboard is the highest-leverage intervention in the whole piece, and it's a UX fix, not a model fix.

> "Today, we see it as a real competitive advantage... it creates a continuous learning loop that improves our models faster than competitors who depend only on automated training."

The reframe from tax to asset only holds if reviews are treated as training data. A review that doesn't feed back into the router or the model is pure overhead.

> "Feedback Loop Poisoning — Vet human judgments before they train the AI, preventing corrupted learning."

A quietly important failure mode: the human reviewers become part of the data pipeline, so inconsistent or gamed review judgments become model corruption. Consensus rounds and audits are quality control for the *humans*, not just the model.

## Key themes

#concept #pattern #tool

- **Tiered review by latency** — match review urgency to risk; most review can be asynchronous.
- **Routing as learned infrastructure** — a trained meta-model deciding who reviews what, with precision asymmetrically optimised at the extremes.
- **Calibrated confidence as the gate** — thresholds are meaningless if confidence scores are miscalibrated; temperature scaling and isotonic regression precede any triage.
- **Reviewer UX as throughput** — context pre-loading bought a 4x review-speedup and up to 10x cases-per-hour.
- **Multi-speed feedback loops** — four time horizons from streaming alerts to quarterly strategy.

## Opinionated take

The strongest part of this piece is its honesty about the audit: 60% of human time on low-impact checks is an indictment most teams never run. The weakness is that every number (85/12/3, 94% router accuracy, 8.3%→2.1% false negatives) arrives without its denominator — what domain, what decision volume, what happens when the router itself drifts on novel inputs it was never trained on. There's a bootstrapping smell here: the router is trained on human-validated decisions, so it inherits the reviewers' biases and freezes the tier boundaries of the era it was trained in. The daily retraining partially answers that, but the piece never confronts the case where the *router* is the thing that needs review.

Also worth noting: this is HITL for ML predictions, not for agent-generated code. The analogy to review-of-agent-work is seductive but imperfect — a prediction has a confidence score; an agent's 400-line refactor often doesn't, and "novelty detection" is much harder over code than over transaction metadata. The transferable idea is the tiering-plus-router shape, not the specific thresholds.

## How it relates

This source strengthens [[Human-in-the-Loop is Tired]] with an engineering answer to Summers' supervision-fatigue diagnosis: if the satisfying part shrank and the exhausting part grew, tiering and context pre-loading are the concrete mechanism for shrinking the exhausting part further — it agrees humans should spend judgment only where it matters and shows the plumbing.

It nuances [[Human Judgment Doesnt Leave the Software Factory — It Relocates]]: Osmani's field reports locate judgment in outer-loop verdicts on agent work; here judgment is relocated into a tiered queue with an explicit router, suggesting the destination is not a role but an architecture.

It complicates [[Oversight Degrades the Overseer]]: where the arXiv paper worries oversight erodes the overseer and proposes batch review as mitigation, this piece independently arrives at batching, pre-loaded context, and throughput tooling as production practice — convergent evidence, though from a domain (prediction validation) where reviewer stakes are lower than in the safety-critical oversight the paper fears.

It extends [[Feedback Loop is All You Need]] with a worked example of multi-horizon loop design: streaming, daily, weekly, and quarterly cycles, each with a named purpose — the taxonomy That piece argues for, instantiated in a running system, including the poisoned-feedback failure mode of humans-in-the-loop-as-training-data.

---
*Sources: [[raw/is-your-human-in-the-loop-actually-slowing-you-down-here-s-what-we-learned]], [[summary/is-your-human-in-the-loop-actually-slowing-you-down-here-s-what-we-learned]]*
*Last updated: 2026-10-03*
