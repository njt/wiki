# FreeToken — Edge-Native MoE Serving

FreeToken is an arXiv proposal for serving Mixture-of-Experts models on the machines people already own. Its core move is to stop treating a personal computer as a small, inadequate GPU and instead treat it as a "unified, elastic inference platform" — one that co-designs model layout, expert residency, CPU–GPU execution, and memory management around the two realities of edge AI: agent workloads change execution pattern continuously, and each machine's CPU/GPU/memory balance is different. The payoff claim is a step change in what fits where: a 35B model on a laptop, 284B on a gaming desktop, and the 753B GLM-5.2 on a single workstation GPU.

---

## Key Quotes

> "treats a personal machine not as a small GPU, but as a unified, elastic inference platform."

This is the paper's reframing, and it is the load-bearing idea. Datacenter serving assumes homogeneous, fixed hardware and builds a static strategy on top. Edge serving cannot: a laptop, a gaming rig, and a workstation expose different bandwidth ratios, so any single offloading recipe is wrong on most of them. The fix is to make the mapping from compute-and-state to resources a *runtime decision*, not a config file.

> "Rather than committing to a fixed offloading strategy, FreeToken continuously maps computation and model state onto the resources actually available."

The direct answer to the fixed-offloading trap. This is the same instinct as [[MiMo-V2.5-Pro-UltraSpeed]]'s model–system codesign, but pointed at the *variable* end of the hardware spectrum rather than a fixed 8-GPU node. Where MiMo/TileRT tuned one engine to one rig, FreeToken tunes continuously to *your* rig.

> "FreeToken supports more than 20 MoE models and real coding and tool-using agents across hardware ranging from an 8GB laptop GPU to a single workstation GPU."

The breadth claim. Support for "more than 20 MoE models" plus real agents signals this is meant as a general platform, not a one-model stunt — the distinction that separates it from a hardcoded engine like [[DS4 (DwarfStar 4)]], which bets on deep integration with a single model.

> "from a 35B model on a laptop to a 284B model on a gaming desktop and the 753B GLM-5.2 on a single workstation GPU."

The concrete headline. The 753B GLM-5.2 figure is the one worth cross-checking: [[GPU Self-Hosting for Coding Agents]] benchmarked GLM-5.2 needing an 8×B200 node for 8 concurrent sessions. If FreeToken genuinely serves it on *one* workstation GPU, that is a different category of result — the difference between "open weights at frontier quality, expensively served" and "frontier quality on hardware you might already own."

> "FreeToken turns open weights into deployable local software, making the machines users already own a practical platform for frontier-scale intelligence."

The thesis restated as consequence. Note the verb: *deployable software*, not a benchmark. The claim is about practical serving, which is exactly the gap [[Local and Open Source Inference]] keeps circling — models are good enough now, but the serving stack that makes them usable on personal hardware is the unsolved layer.

---

## Key Themes

- #tool **Bandwidth-adaptive execution** — instead of a fixed CPU/GPU offloading split, the system continuously remaps computation and model state to the resources actually available, machine by machine.
- #concept **Edge-native vs. datacenter-native serving** — the paper's fundamental stance: personal machines are not degraded datacenter nodes; they have their own heterogeneous resource profile that a serving stack should adapt to rather than fight.
- #concept **Expert residency** — for MoE models, the core edge problem is which experts live in GPU memory, which in CPU memory, and when they migrate, since only a few experts activate per token.
- #concept **Agentic state reuse** — treating the evolving state of a coding/tool-using agent as a first-class resource to be managed, not an afterthought.
- #pattern **Model–system codesign at the edge** — the same methodology MiMo/TileRT applied to a fixed node, applied to heterogeneous personal hardware.

---

## Critical Analysis

**The framing is right, and it is genuinely new.** Most local-serving discussion assumes the datacenter playbook shrunk down — vLLM/SGLang on a 3090, static offloading, batch throughput as the metric. FreeToken's two premises (workloads change shape; machines differ from each other) are true and under-served. "Bandwidth-adaptive" is the right name for the axis that actually differs between an M-series Mac, a 8GB laptop dGPU, and a workstation: the CPU↔GPU↔memory bandwidth balance.

**The evaluation is invisible from the abstract.** This is the paper's fatal weakness as a source: the fetch captured only the arXiv abstract — no authors, no latency numbers, no tokens-per-second, no comparison against the baselines it implicitly claims to beat (vLLM/SGLang offloading, Ollama, llama.cpp). "Supports more than 20 MoE models" and "753B GLM-5.2 on a single workstation GPU" are assertable in an abstract and unverifiable from it. The 753B-on-one-workstation claim directly contradicts the best public benchmark we have — [[GPU Self-Hosting for Coding Agents]] put GLM-5.2 on 8×B200 — so either FreeToken is doing something genuinely better, or the claim is about fitting the model in memory and running it, not serving it *well* under agentic load. Those are different things, and the abstract does not tell us which.

**The honest verdict is "directionally important, evidence pending."** If real, FreeToken is the missing serving layer that [[Local Models in Mid-2026]] gestures at — the systems answer to MoE's "heavy on capacity, light on bandwidth" profile that Coles mapped onto unified memory. But until the paper's benchmarks can be read, it sits alongside [[AirLLM]] and [[Petals — Decentralized LLM Inference]] as a promising claim about edge serving that has not yet been independently confirmed.

---

## Cross-References

- [[Local and Open Source Inference]] — the hub page: FreeToken is the serving-stack candidate for its central thesis that serious work no longer requires a cloud model
- [[GPU Self-Hosting for Coding Agents]] — the benchmark to beat; its 8×B200 GLM-5.2 result is the number FreeToken's single-workstation claim implicitly contradicts
- [[Local Models in Mid-2026]] — the MoE/bandwidth/unified-memory analysis FreeToken operationalises into a running system
- [[MiMo-V2.5-Pro-UltraSpeed]] — model–system codesign on a fixed node; FreeToken extends the methodology to heterogeneous personal hardware
- [[DS4 (DwarfStar 4)]] — the single-model-deep-integration opposite pole to FreeToken's 20+-model platform breadth
- [[Inference Cost Napkin Math]] / [[Theoretical LLM Inference Bottlenecks]] — the bandwidth-bound decode analysis that "bandwidth-adaptive" is answering

---

*Sources: [[raw/2608-16157]], [[summary/2608-16157]]*
*Last updated: 2026-08-22*
