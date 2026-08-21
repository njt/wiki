# GPU Self-Hosting for Coding Agents

The most thorough public benchmark of self-hosted inference for coding agents: aistack (imec) ran ~100 runs of 64 SWEBench Pro tasks across four hardware tiers (DGX Spark → 8×B200), four open-weight models, and commercial APIs, measuring concurrency limits, task completion time, and cost. The data is the best available answer to "should we buy GPUs or keep paying for API tokens?" — and the answer is "it depends on your model, your utilization, and which of quality/speed/cost you're willing to sacrifice."

---

## The Setup: Four Hardware Tiers, Four Models

aistack paired each hardware tier with the best open-weight model that fits in its memory budget:

| Hardware | Model | Inference Engine | Comfortable Concurrency |
|---|---|---|---|
| DGX Spark (desktop, 128GB) | Qwen3.6-35B-A3B | vLLM | 1 (and still slow) |
| 1× H200 (141GB) | Qwen3.6-35B-A3B | vLLM | 32 |
| 4× H200 (NVLink) | DeepSeek-V4-Flash | vLLM (TP=4) | 32 |
| 8× B200 | GLM-5.2 | SGLang (TP=8) | 8 |
| 8× B300 | Kimi K3 (update) | SGLang | 16 |

The K3 update (July 29) used B300s with 288GB HBM per GPU (2.3TB/node) — ~20% more expensive than the B200 setup but necessary because K3's 1.4TB weights wouldn't fit in the B200's 1.5TB HBM with room for KV cache.

## Key Findings

### Concurrency: The DGX Spark Is a Toy, GLM-5.2 Is Painfully Narrow

> "Your DGX Spark is really only for single use. It will fit one developer on a good day with a small model and a moderate token pool."

The DGX Spark couldn't push past 1–2 concurrent users without "an avalanche of time-out errors." Despite LinkedIn headlines about "frontier intelligence on a desktop," we're not there yet for agentic workloads.

At the other extreme, GLM-5.2 on 8×B200 — the most powerful hardware tested — only comfortably serves **8 concurrent sessions** before tasks take 3× longer than the Claude Code (Opus 4.8 API) baseline. Stacking eight B200s to serve eight developers is a brutal ratio.

That conclusion is worth holding against [[FreeToken — Edge-Native MoE Serving]], which claims the same 753B GLM-5.2 can run on a *single* workstation GPU via bandwidth-adaptive CPU–GPU execution. If the claim survives scrutiny, it would overturn "frontier quality needs a B200 rack" — but the abstract offers no latency or concurrency numbers, so it cannot yet be weighed against the 8-concurrent-session ceiling measured here.

The sweet spot is the middle: a single H200 running Qwen3.6 comfortably handles **32 concurrent sessions**, and 4×H200s with DeepSeek-V4-Flash does the same at higher quality.

### The Collapse Problem: vLLM Defaults Break Under Load

> Both Qwen3.6 on H200 and DeepSeek-V4-Flash on the HGX H200 take a nose dive after concurrency 48 and 64.

This is the most alarming finding and the one the article is most cagey about. The inference engine (vLLM) with default parameters catastrophically degrades under load — throughput doesn't plateau, it *collapses*. The authors finger vLLM's prefill/decode scheduling and KV-cache management as the culprits, plus speculative decoding helping at low concurrency but hurting at high concurrency. They promise a follow-up on tuning this away, but the implication is clear: **out-of-the-box vLLM is not production-ready for high-concurrency agent workloads.** You need an inference engineer.

### Cost: Three Completely Different Answers

The cost comparison per 64-task run, using the optimal (not necessarily comfortable) concurrency for each setup:

| System | API Cost | Buy (5yr deprec.) | Buy (30% util) | Rent |
|---|---|---|---|---|
| Qwen3.6 / 1×H200 | $57.67 | $0.43 | $1.43 | $1.62 |
| DeepSeek-V4-Flash / 4×H200 | $2.13 | $1.90 | $6.33 | $7.19 |
| GLM-5.2 / 8×B200 | $92 | $13.84 | $46.12 | $71.23 |
| Opus 4.8 (API baseline) | $98 | — | — | — |

The story is three different economics in one table:

- **Qwen3.6:** Renting is 35× cheaper than API. A no-brainer to self-host if Qwen3.6's quality is acceptable.
- **DeepSeek-V4-Flash:** Self-hosting is *more expensive* than DeepSeek's own API. The API's cache pricing — up to 98% of input tokens cached for coding agents — beats even the raw hardware cost. A 4×H200 needs 89% utilization (24/7/365) just to break even.
- **GLM-5.2:** Self-hosting beats the frontier API at just 15% utilization. At rental rates, frontier-quality tasks cost $1.11 apiece — cheaper than either Anthropic or the GLM API.

The counterintuitive result: **buying GPUs makes more economic sense at the high end (near-frontier quality) than the low end**, because the API markup on frontier models is so large that even poorly utilized hardware beats it, while DeepSeek's aggressive cache-friendly pricing makes its API cheaper than running the same model yourself.

### Model Quality: K3 Is the Outlier

Resolution rates on the 64-task SWEBench Pro subset:

| Model | Resolution Rate |
|---|---|
| Kimi K3 (8×B300) | 86.4% |
| GLM-5.2 (8×B200) | 62.5% |
| Opus 4.8 (API) | 62.5% |
| DeepSeek-V4-Flash (4×H200) | 39.1% |
| Qwen3.6 (1×H200) | 35.4% |

K3's 86.4% is so far above everything else that the authors themselves flag it: SWEBench Pro tasks may have been in K3's training data. Treat with salt. But GLM-5.2 genuinely matching Opus 4.8 on this benchmark is the real story — open-weight models at frontier quality are here, just expensive to serve.

The drop from GLM-5.2 (62.5%) to DeepSeek-V4-Flash (39.1%) is steep. You're paying for that quality gap either in hardware (8×B200 vs 4×H200) or in task failure rate.

## Critical Analysis

### The DGX Spark Result Is the Most Honest Thing in AI Hardware Right Now

"1 user, and still slow." In an industry where every hardware vendor claims to run frontier models on a desktop, aistack just published benchmark data showing the DGX Spark can't handle more than one developer and even then is 3× slower than API. This is the kind of honesty that makes the whole article trustworthy.

### Utilization Is the Invisible Tax

Published enterprise GPU utilization is 15–22%. A well-run deployment rarely exceeds 25–35%. That means **65–85% of the hardware you bought is idle at any moment** — and you paid for all of it.

The article acknowledges this but doesn't fully explore the implication: the utilization problem means self-hosting is structurally disadvantaged against API pricing for bursty workloads like coding agents. The one counter-trend they identify — automated agents running overnight — is real but nascent. Most teams aren't there yet. [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]] is a single-user data point for that counter-trend: a budget 2×RTX 5060 Ti box running Qwen3.8-27B completed a real three-repo bug hunt in ~10 minutes unattended — and the metric that mattered was wall-clock to a correct result, not the throughput/concurrency this benchmark optimizes.

This rhymes with [[Inference Cost Napkin Math]], where duty cycle is "the 5× multiplier nobody measures," and with the broader cloud economics lesson that killed on-premise datacenters: utilization is everything.

### The Hybrid Argument Is Buried in the Data

The article presents a three-way choice (API vs. rent vs. buy), but the data screams for a fourth: **tiered routing**. Use DeepSeek-V4-Flash API for the 98% of cached, routine coding tasks (at $2.13 per 64-task run — essentially free), and rent B200 hours for the hard problems that need GLM-5.2 quality. This is the architecture [[Model Routing Is Simple Until It Isn't]] warns is harder than it looks, and [[Thrifty (Tiered Delegation for Claude Code)]] implements for Claude models specifically.

### The Methodology Appendix Is the Most Valuable Page

The article buries its methodology in an appendix, but it's the best part. The decision to retain setup/evaluation phases (so agents occasionally idle like real developers grabbing coffee), the use of sustained throughput rather than peak, the honest treatment of variance in the collapse behavior, and the reproduction of results against Harbor to validate — this is how benchmarking should be done. The 64-task SWEBench Pro subset is small enough to be practical but large enough to be meaningful. The ~100 total runs across configurations is real effort.

### What's Missing: Depreciation Risk and Model Improvement Rate

The cost analysis uses 5-year straight-line depreciation, but GPU hardware depreciates faster than that in practice, and open-weight model improvement means today's "frontier-quality" model is next year's "budget tier." A GPU bought today needs to earn back its cost before the models it runs become obsolete. The article doesn't model this, and it's probably the most important variable for anyone writing a purchase order. See [[Open models lag state-of-the-art closed models by 4 months]] and [[NVIDIA B300 vs H200 GPU Analysis]] for the hardware side of this equation.

### The Inference Engineer Bottleneck

The collapse problem reveals a hidden constraint: self-hosting at scale requires an inference engineer who can tune vLLM/SGLang parameters for your specific workload. This isn't an "install and forget" proposition. The authors promise a follow-up on tuning, but for now, the message is: **the defaults will fail you under load, and fixing it requires expertise most teams don't have in-house.** This is the same dynamic [[In-House LLM Serving at Netflix]] documented — the gap between vendor tooling and production reality is where the real engineering lives.

## Key Themes

- `#economics` — Three different cost curves depending on model: API wins for DeepSeek, self-host wins for Qwen and GLM-5.2. Utilization is the deciding variable.
- `#hardware` — DGX Spark → H200 → 4×H200 → 8×B200 → 8×B300: a complete hardware ladder with benchmarked concurrency limits
- `#benchmark` — 64 SWEBench Pro tasks × ~100 runs × multiple concurrencies = the most rigorous public self-hosting benchmark for coding agents
- `#inference-engineering` — vLLM defaults collapse under load. Speculative decoding helps low concurrency, hurts high. Inference tuning is a required skill.
- `#open-weights` — GLM-5.2 matches Opus 4.8 on this benchmark. K3 exceeds everything (with contamination caveat). The gap is closing.
- `#tradeoff` — Quality × Speed × Cost: pick two, and even then the answer changes per model

## Recommendations

1. **Don't buy GPUs to save money on API bills unless you can keep them busy.** At honest utilization, the economics are brutal.
2. **If you're spending >$50K/year on frontier API tokens, rent a B200 rack.** The 15% utilization breakeven is achievable, and you get frontier-quality at $1.11/task.
3. **For most teams, the right answer is DeepSeek-V4-Flash API for routine work + rented GPUs for the hard 20%.** DeepSeek's cache-friendly pricing makes self-hosting DeepSeek actively more expensive.
4. **Budget for an inference engineer if you go the self-host route.** vLLM defaults will fail you under agentic load, and tuning requires expertise.
5. **Measure your own workload.** The article's numbers are the best public data available, but your concurrency patterns, model preferences, and utilization will differ.

---

*Sources: [[raw/gpu-self-hosting]]*
*Last updated: 2026-08-01*
