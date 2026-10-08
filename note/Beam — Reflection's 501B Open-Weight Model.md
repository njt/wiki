# Beam — Reflection's 501B Open-Weight Model

Reflection AI's first open-weight release: a 501B-total / 23B-active sparse MoE trained on 23.8T tokens and pushed hard through high-compute RL (100M+ rollouts, 10.5K GB300 GPUs, four weeks). The pitch is not frontier-capability leadership but *efficiency* — GLM 5.2-class results at 3–4× less inference compute — plus Apache 2.0 weights and the full run/eval/fine-tune stack. The post is as much an RL infrastructure paper as a model card.

---

## Key quotes

> "Beam advances the Western open-weight frontier and is competitive with larger open models like GLM 5.2 and approaching Qwen 3.8-Max on coding and agentic tasks. Where frontier open models like Kimi K3 remain ahead on raw capability, Beam's advantage is efficiency at inference time."

Careful positioning: Reflection concedes raw capability to Kimi K3 and sells intelligence-per-token instead. That is a workhorse strategy, not a flagship strategy — aimed at the enterprise coding market where cost per resolved task is the buying criterion.

> "Across our evaluation suite, capabilities continued to improve as we increased RL compute, with no sign of a plateau."

The claim that keeps the whole post honest: RL scaling as a still-open axis. But it is self-reported, pre-release, and benchmark selection is theirs — the promised technical report will be the actual test.

> "We developed new algorithms to maintain stable learning under these conditions... even when learning from interactions generated more than a day earlier."

Fully asynchronous policy gradients with version-tagged tokens is the genuinely interesting technical contribution. Day-old experience being usable at all says the stall-tolerance problem at 110K concurrent rollouts is severe — and that Reflection solved it well enough to keep going.

> "About 95% of raw Internet tokens are eliminated through parsing, deduplication, and curation... conventional techniques would have missed roughly 1.8 trillion high-quality tokens we retain."

Data curation as competitive moat. The 87% figure for curated web-code tokens missed by conventional filters is the number that matters for a coding model — and implicitly a dig at generic OSS data pipelines.

> "During a phase of training on reasoning, software engineering, and terminal tasks, we saw consistent gains in browsing despite the absence of browsing tasks from the RL mixture."

The generalization evidence is the most believable part of the capability story: transfer between tool-use domains, plus organically discovering LLM-querying and OCR APIs, is harder to fake with benchmark tuning than leaderboard deltas.

## Key themes

#concept — high-compute RL as a scaling axis, asynchronous policy gradients, multi-teacher on-policy distillation
#tool — Beam (501B MoE, Apache 2.0), reasoning-effort parameter
#pattern — 1M synthetic RL environments with iterative difficulty/quality filtering; reward forecasting as eval
#person — Reflection AI (Misha Laskin / Ioannis Antonoglou's lab), positioning against DeepSeek-style open releases

## Analysis

Three things stand out.

**The RL infrastructure is the actual product.** The model spec (sparse MoE, interleaved attention, aux-loss-free balancing with cosine decay) is competent engineering built on published DeepSeek-era ideas. The distinctive asset is the platform: 170K concurrent sandboxes across two clouds, 12-second hierarchical weight distribution over RoCE+NVLink, 99.99% trainer packing, 71 absorbed inference failures, per-token records for verifier-exploit audits. That is a decade of distributed-systems work compressed into one blog post, and it is the part competitors cannot copy from weights alone. The "replayable records" and independent judge re-screening for reward hacks also quietly acknowledge that at 100M rollouts, verifier exploits are an everyday operational problem, not a research curiosity.

**The efficiency framing is a market thesis, not just a spec sheet.** "More intelligence per token" is aimed squarely at enterprises running coding agents at volume, where inference cost dominates and a GLM-5.2-class model at a third of the compute changes unit economics. Whether the 3–4× figure survives independent evaluation matters less than the fact that open-weight competition is now being fought on serving cost, not just benchmark scores.

**The gaps are the usual ones.** No benchmark table survives a blog summary (figures are promised in the technical report), "final red-teaming" means nobody independent has scored it yet, and the safety section — while unusually substantive (adversarial jailbreak iteration, forecasting r = 0.79 beating the BoN ceiling as a reward-design tool) — is still self-graded. The commitment to open-source their safety evaluations is the right gesture and exactly the kind of claim a wiki should hold open until it ships.

## Related pages

- [[GLM-5.2 Is the Step Change for Open Agents]] — Beam is the direct answer to the open-weight moment Lambert described; it benchmark-matches GLM 5.2 while attacking it on serving cost rather than license or latency, complicating the "one step-change model" narrative with a second Western contender.
- [[How Far Behind Are Open Models]] — This release is a data point for that page's lag measurement: if the efficiency claims hold, the open-closed gap narrows further, but the pre-release, self-reported status means the lag clock should not be reset until the technical report lands.
- [[MiMo-V2.5-Pro-UltraSpeed]] — Xiaomi and Reflection are converging on the same thesis from opposite ends: model–system codesign where inference efficiency, not parameter count, is the deliverable; Beam's MoE stability recipe is the training-side twin of MiMo's serving-side codesign.
- [[Local Models in Mid-2026]] — An Apache 2.0 501B MoE with 23B active and a shipped fine-tuning stack extends that survey's five engineering advances into heavyweight territory: the "open weights competitive for everyday work" trend now has an enterprise workhorse candidate at the top of the size range.

---
*Sources: [[raw/introducing-beam]], [[summary/introducing-beam]]*
*Last updated: 2026-10-08*
