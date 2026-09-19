# Bonsai 2 27B

PrismML's second-generation Bonsai compresses the newer Qwen3.8 27B to ternary {−1, 0, +1} weights with FP16 group-wise scaling — 1.76 effective bits per weight, 5.9 GB, more than 9x smaller than full precision — while retaining 98.2% of the base model's aggregate benchmark performance (up from ~95% for the first generation). The release pitches that retention as landing exactly where compounding errors hurt most: coding agents, tool use, multimodal workflows, and long-horizon tasks. Apache 2.0, 262K context, multimodal, custom low-bit kernels for CUDA and MLX.

---

## Key Quotes

> Against its full-precision counterpart, Ternary Bonsai 2 27B is more than 9x smaller while retaining 98.2% of aggregate benchmark performance. At this level of retention, compression becomes a deployment unlock: nearly the same capability, in a footprint that can run in far more places.

The generational delta is the real story: 95% → 98.2% retention, on a stronger base. But "aggregate" is doing quiet work here — a weighted average can hide exactly the per-task collapses that made the first Bonsai's low-bit variants awkward for agents. The number to want is the per-benchmark table, and the announcement defers all of it to the whitepaper.

> Coding agents, tool-use systems, multimodal workflows, and long-horizon tasks are particularly sensitive to model degradation because small errors can compound over many steps. Bonsai 2 27B preserves much of the full-precision model's performance in exactly these areas.

The diagnosis is correct and matches what the first release showed — agentic and instruction-following were its weakest axes. But "preserves much of" is an assertion, not a table. "Near-lossless" is a claim the whitepaper has to earn benchmark by benchmark, especially for agentic scores.

> On an RTX 4090, Ternary Bonsai 2 27B consumes just 0.714 mWh/token, making it 40% more energy-efficient than an 8B model running in full-precision.

The quiet headline. A 27B model beating an 8B full-precision model on energy per token inverts the usual bigger-means-more-power intuition and makes energy a first-class deployment metric alongside memory. The comparison baseline is chosen to flatter (8B FP16 vs 27B ternary), but the direction is real: the meter sees bits per weight, not parameter count.

> The question will increasingly be not just how capable a model is, but how much useful intelligence can be delivered within a given memory, compute, and power budget.

The continuation of the intelligence-density framing from the first release, now generalized beyond phones to the whole stack. It reframes model quality as a ratio, and it is the right ratio for a market where DRAM prices have doubled — which is also why a 9x footprint reduction is commercially well-timed rather than merely technically impressive.

## Key Themes

- **#concept** **Intelligence density, generalized**: The first release defined intelligence per GB for phones; this release extends the metric to memory, compute, and power budgets across devices, workstations, and datacenters. Same idea, bigger stage — and the datacenter implication (more users per GPU) is where the economics actually bite.

- **#pattern** **Same-deployment-point upgrades**: Hold the footprint constant (5.9 GB, same as generation one), swap in a stronger base model, and measure progress as retention at fixed deployment cost. This is performance-per-watt thinking applied to model releases — the right way to compare quantization generations, and a pattern worth watching across the industry.

- **#concept** **Near-lossless as a threshold effect**: Below ~95% retention, compression is a visible tradeoff you design around; near 98%, it starts to become transparent — the deployment decision stops being about capability and starts being purely about footprint. If the claim holds per-benchmark, the interesting question shifts from "how much did it degrade?" to "why run full precision at all?"

- **#tool** **Hybrid local/cloud orchestration**: The same pitch as the first release, now with more capability behind it — local handles sensitive or high-frequency work, cloud gets selective escalations. Repeatable positioning across two releases suggests it is the company's actual product thesis, not launch-copy filler.

## Critical Analysis

**The sustainability test from the first release gets a first affirmative answer.** The obvious question about Bonsai 27B was whether extreme quantization was a one-off achievement or a repeatable pipeline. Bonsai 2 answers for one cycle: newer base (Qwen3.6 → Qwen3.8), higher retention (95% → 98.2%), identical 5.9 GB ternary footprint. That is the pattern that would make Bonsai a product line rather than a stunt — retarget the compression at each new base model as the frontier moves. One cycle is one data point, but it is the right data point.

**Read the headline against their own previous best, not against full precision.** "9x smaller" was already true of Bonsai 1. The like-for-like comparison is 5.9 GB → 5.9 GB: zero size progress, all capability progress. That is a legitimate story — but notice what did not advance this time: the phone. The first release's headline was the 3.9 GB 1-bit variant fitting an iPhone's ~6 GB usable budget; this release ships ternary only, so the pocket frontier is unchanged while the laptop-and-edge frontier moved up. Also notable: ternary weights with FP16 group-wise scaling is a more precise — and slightly less absolutist — description than generation one's "no higher-precision escape hatches" framing. Group-wise scaling factors are standard, negligible in size, and already accounted for in the 1.76 effective bpw figure, but "effective bits per weight" is the number to read, not the marketing bit-width.

**Throughput went backwards on the like-for-like comparison.** Generation one's ternary model did 87 tok/s on an M5 Max; generation two does 46.8 tok/s on the same machine — roughly half. On RTX 5090 the new model does 143 tok/s, but the comparable generation-one figure there was for the smaller 1-bit variant. The most plausible reading is that the stronger Qwen3.8 base costs decode speed at current kernel maturity. For agent loops, where many sequential calls dominate wall-clock time, a halved decode rate is a real cost — the intelligence-density gain partly came out of throughput, and kernel work will need to claw it back.

**The aggregate hides the variance that matters.** The first release published per-benchmark numbers and let you see agentic degradation; this one publishes one number and a whitepaper pointer. The claim that retention holds "exactly where it matters" is plausible — the compression pipeline is now proven — but near-lossless is a per-task property, not an average. This wiki's own field reports ([[Qwen3.8 27B Hardware Tests]]) show the full-precision base itself trails at long context, so the 262K window inherited by Bonsai 2 deserves the same suspicion: retention measured on short benchmarks says little about behavior at 100K+ tokens.

**The energy number is the sleeper.** 0.714 mWh/token on an RTX 4090 is the kind of metric that matters more as local inference moves to always-on background assistants — the article's own use case. Combined with the hybrid-orchestration pitch, the strategic picture is coherent: bits per weight is becoming the axis on which local AI competes, and PrismML is trying to own that axis the way [[Bonsai 27B]] first staked it out. The MLX + CUDA dual-platform release also keeps the Apple-side ecosystem ([[mlx-dspark]]'s speculative decoding for the Bonsai family) immediately applicable to the new weights.

---

*Sources: [[raw/bonsai-2-27b]], [[summary/bonsai-2-27b]]*
*Last updated: 2026-09-19*
