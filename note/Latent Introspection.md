# Latent Introspection

A 32B open-weight model (Qwen2.5-Coder-32B-Instruct) can detect when a concept was injected into its earlier context via a steering vector — and say *which* concept — but only if you know where to look: the signal is visible in middle-layer logits and nearly absent from sampled outputs. This is the open-weights replication of Anthropic's introspection work, and its most interesting claim is about evaluation, not consciousness: capabilities can exist that behavioral testing cannot see.

---

## The Setup

The design isolates introspection from trivial explanations:

1. Apply a steering vector (PCA-extracted from contrastive prompts, via repeng) to middle layers (21–42) while generating the KV cache for turn one.
2. Remove the vector. Ask the model about the injection.

Because the injection is gone by question time, a "yes" answer can't be a reaction to ongoing perturbation — it must come from representations cached in prior tokens. Control conditions (always-yes/always-no questions, norm-matched Gaussian noise instead of a concept vector) barely move; the noise shift is two orders of magnitude smaller than the concept-vector shift.

## The Signal Is Hidden in Three Ways

- **Hidden in layers.** Logit lens shows detection emerging around layers 50–60 and attenuating in the final layers. The model's *answer* is "no" either way; the model's *knowledge* is in between.
- **Hidden in prompt framing.** Baseline: P("yes") moves 0.2%→0.3%. With a Pro-Introspection Document (KV-cache information flow + Lindsey's abstract): true positives jump to 39.9% at +0.6% false positives. Vague/Poetic framings reach 84% balanced accuracy — and beat the "Correct Target" framing that literally describes activation injection, which the authors hypothesize trips post-trained refusal/denial behavior.
- **Hidden in sampling.** Reading logits directly is orders of magnitude cheaper than estimating such small probability shifts by sampling — which is exactly why sampling-based evaluation would have missed this forever.

Concept identification (which of nine concepts: cats, bread, truth, death...) reaches 1.66 bits MI, 52% of theoretical max, correlated with detection sensitivity (r = 0.68). Generic noise can't do that.

## Why It Matters

> "Behavioral evaluation alone may not be sufficient to rule out capabilities."

That's the load-bearing sentence. If capabilities can sit latent — detectable only in probability distributions, intermediate layers, or under specific elicitation prompts — then every eval suite built on sampled outputs has a systematic blind spot. And the mechanism of the hiding matters for safety: the authors offer three untested hypotheses for the late-layer suppression (post-training penalizing self-awareness claims, pretraining dynamics, distribution shift), and the "training models to be honest is complicated if existing mechanisms suppress such reports" point cuts deep.

For agent builders the flip side is concrete: activation steering is a commonly proposed alignment defense, and this paper says you cannot assume a model fails to notice it. Anything that touches the KV cache or residual stream may leave a readable trace the model can, in the right context, report on.

## What They Don't Claim

The scope discipline is exemplary. They claim "intervention reporting" — accurate report on internal states with privileged access — not second-order metacognition. They engage concurrent skeptics (Singh et al. 2026: models reproducing detection can't distinguish activation interventions from matched input manipulations, suggesting generic anomaly detection) and accept the limit that concept readouts may show steering effects *persist*, not that they are *recognized*. Prompt-dependence is itself a finding they don't fully understand — the Poetic framing outperforms the technically correct one, which is either refusal-triggering or a sign that "what seems salient" is a more natural internal representation than "what was done to my activations."

## Cross-links

This strengthens [[Jacobian Lens]] and [[Verbalizable Representations Form a Global Workspace in Language Models]]: both Anthropic techniques read what internal activations are "disposed to say" without sampling, and this paper shows the same philosophy — logit-level readouts — catching a capability sampling misses entirely. The two papers together make a strong case that "what the model says" and "what the model represents" are different measurable objects, with the gap between them being where safety surprises live.

It nuances [[They're Made Out of Weights]]: Leiter's dialogue leans on the "just weights, nothing to see here" framing; a model that can report on modifications to its own earlier activations makes "just weights" feel less like a settling of accounts and more like an open question.

It also connects to [[Recent Developments in LLM Architectures]] in an unexpected way: the KV cache is usually discussed as an inference-cost engineering object, but this paper treats cached representations as the substrate of self-knowledge — the model's memory of its own prior turn is literally where the introspective signal lives.

#concept #pattern

---
*Sources: [[raw/2602-20031v3]], [[summary/2602-20031v3]]*
*Last updated: 2026-11-26*
