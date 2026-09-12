# Hy3

Tencent's 295B MoE open-source model (Apache 2.0) with 21B active parameters, positioned as a coding-and-agent workhorse that rivals flagship models with 2–5× the parameter count. Ships with MTP speculative decoding, 256K context, first-class vLLM/SGLang deployment, and an FP8 quantized variant.

---

## Key Quotes

> Hy3 "outperforms similar-size models and rivals flagship open-source models with 2-5x parameters."

The quiet thesis of this release: parameter count stopped being the scoreboard. At 295B total / 21B active, Hy3 is smaller than the megamodels but its SWE-bench Verified score of 78 puts it in a tier where the metric that matters is agent reliability, not parameter count. This is the same ratio-game [[Local Models in Mid-2026]] identifies as the number that actually matters for real workloads.

> On SWE-Bench Verified, accuracy variance across scaffoldings like CodeBuddy, Cline, and KiloCode stays within 4%.

Tool-call stability across harnesses is the claim that matters most. Every model release boasts benchmark scores; almost none report variance across agent scaffolds. A 4% spread means Hy3's tool-use behavior is robust to the scaffolding choices that typically make or break agent performance — the same reliability story [[Step 3.7 Flash]] pitches with "less drift, fewer broken toolcalls."

> The hallucination rate dropped from 12.5% to 5.4%, and commonsense error rates fell from 25.4% to 12.7%.

Tencent is publishing internal pre/post metrics on hallucination, which is unusual and useful. The 5.4% hallucination rate is a specific, falsifiable claim — and one most model cards don't make. The commonsense error halving suggests the post-training regime (joint SFT + RL) is doing real work on factual grounding, not just benchmark optimization.

> "Hy3 also generalizes across different agent scaffoldings."

This claim about scaffolding-agnostic performance echoes an emerging design principle: the model should be the stable component, not the harness. It's the inverse of [[Honey I Shrunk the Coding Agent]], which proved the harness matters more for small models — Tencent is arguing that at 21B active parameters, the model is strong enough that the harness becomes secondary.

---

## Key Themes

**#concept — MoE density ratio as the new parameter count.** 295B total / 21B active gives Hy3 a ~14:1 ratio. This is more aggressive than [[Step 3.7 Flash]] (196B/11B = ~18:1) but the active parameter count is nearly double, which explains the higher benchmark scores. The ratio, not the total, is what dictates deployment cost.

**#tool — Agent reliability as the new benchmark.** SWE-bench Verified 78, SWE-bench Pro 57.9, Deep SWE 28 — these aren't abstract reasoning scores, they're measures of whether the model can actually complete real software tasks. The 4% cross-scaffolding variance is the stat that should make engineering teams pay attention.

**#pattern — MTP as table stakes.** Multi-token prediction with 1 MTP layer and `num_speculative_tokens 2` — this is now standard equipment for large models, not a differentiator. [[JetBrains Mellum2]] and [[MiMo-V2.5-Pro-UltraSpeed]] both ship MTP heads. The innovation has moved from "does it have MTP?" to "how many MTP layers and how many speculative tokens?"

**#pattern — FP8 quantization at launch.** Shipping an FP8 variant (Hy3-FP8) alongside the BF16 model is becoming the norm rather than an afterthought. Combined with AngelSlim (Tencent's compression toolkit), this signals that quantization is now a first-class deployment concern, not a community hack.

---

## Critical Analysis

**The scaffolding-agnostic claim is the most important thing on the page — and the least verified.** A 4% variance across three scaffolds (CodeBuddy, Cline, KiloCode) is genuinely impressive if it holds. But it's reported on SWE-Bench Verified, which has known saturation issues. I'd want to see the same stability on SWE-Bench Pro or Deep SWE before believing the model has truly decoupled from its harness. The history of agent benchmarks is littered with models that looked scaffold-agnostic on easy tasks and fell apart on hard ones.

**The blind evaluation (2.67/4 vs GLM-5.1's 2.51/4) is clever marketing dressed as methodology.** 270 experts is a real sample size. But "blind evaluation" without published methodology, inter-rater reliability, or task distribution is a vibe, not a measurement. The specific claim — largest advantage in frontend, data & storage, CI/CD — is where the real information lives, and it's frustratingly thin.

**Hallucination reduction from 12.5% to 5.4% is the right metric reported the right way.** Pre/post on a specific, defined measurement. More model cards should do this. The lingering question: what's the measurement methodology? Is this factual hallucination on a knowledge benchmark, or tool-call hallucination in agent loops? The two have different root causes and different fixes.

**Hy3 sits in an increasingly crowded lane.** [[Step 3.7 Flash]] (196B/11B), [[Cohere North Mini Code]] (30B/3B), [[JetBrains Mellum2]] (12B/2.5B), [[MiMo-V2.5-Pro-UltraSpeed]] (1T total) — there are now at least five serious open-weight MoE models competing for the "agent workhorse" position. Hy3's differentiator is the combination of scale (largest active params in the open MoE tier), Apache 2.0 licensing, and the scaffolding-stability claim. Whether that's enough depends on whether the stability claim replicates.

**The Tencent model ecosystem is becoming a thing.** Hunyuan → Hy3 Preview → Hy3, with AngelSlim for compression and dedicated deployment guides for vLLM and SGLang. This isn't a one-off model drop; it's a platform play. The question is whether the open-source community adopts Tencent's stack or just strips the weights and runs them through existing infrastructure. The platform now spans modalities too: [[AuK]], Tencent's 1.5B MIT speech foundation model, shipped the same month with the same Day 0 SGLang support pattern — text/code and speech under one open-weight umbrella.

---

*Sources: [[raw/hy3]]*
*Last updated: 2026-07-11*
