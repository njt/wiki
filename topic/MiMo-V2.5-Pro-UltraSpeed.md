# MiMo-V2.5-Pro-UltraSpeed

Xiaomi's MiMo team and TileRT jointly announce a 1-trillion-parameter MoE model running at 1000+ tokens/s on a single 8-GPU commodity node — achieved not through specialized hardware (Cerebras/Groq) but through extreme model-system codesign: selective FP4 quantization of MoE Experts, DFlash block-level speculative decoding, and TileRT's persistent-kernel inference engine. Launched June 8, 2026 with a limited two-week application-based trial (June 9–23). Open-source FP4-DFlash checkpoint available on HuggingFace.

---

## Key Quotes

> "When a model is fast enough, it ceases to be a tool you wait on and becomes an extension of your own thinking: responding in real time, iterating in an instant, collaborating without friction."

The product frame is exactly right. Speed isn't a feature — it's the difference between a tool and a collaborator. The same transition happened with IDEs (instant feedback replaced batch compilation), with search (Google replaced library stacks), and with messaging (Slack replaced email). Every order-of-magnitude latency reduction creates qualitatively different usage.

> "Speed itself begins to transmute into intelligence. Previously, when facing a hard problem, you could only 'wait for one answer and pray it's correct.' Now, within the same wall-clock time, the model can run dozens of reasoning paths in parallel (Best-of-N / Tree Search), automatically verifying and self-correcting in the background."

This is the core insight, and it's underappreciated. Inference speed isn't just about UX — it's a compute budget you can spend on **reasoning depth**. At 1000 tps, you can run 10 parallel reasoning chains and still get results faster than a single chain at 100 tps. Speed compounds into intelligence.

> "Through this extreme Codesign, we achieved 1000+ tokens/s output from a 1T model using just a single standard 8-GPU commodity node."

The "commodity GPU" claim is the headline here. Cerebras and Groq have done extreme speeds on custom silicon — MiMo/TileRT did it on hardware you can buy. This makes the breakthrough accessible in a way wafer-scale integration never will be.

> "At 1000 tokens/s operating frequency, each operator's lifecycle is compressed to microseconds, and the 'operator boundaries' of traditional inference systems become the core bottleneck — every operator launch, hardware synchronization, and global memory round-trip fractures the execution flow at the microsecond scale, exposing visible 'Execution Gaps.'"

A concise diagnosis of why traditional inference stacks can't just be "sped up" — at microsecond timescales, the framework overhead becomes the bottleneck, not the compute. TileRT's solution (persistent kernels, warp specialization) is effectively building a real-time OS for GPU inference.

> "When this power is brought to surgical assistance and medical imaging analysis in life-or-death situations, AI speed is no longer just a metric of efficiency — it becomes a chip in the race against death. On the operating table, every second AI saves in completing lesion analysis and risk prediction gives the surgeon one more degree of freedom."

This is the most emotionally effective passage in the announcement. The "race against death" framing connects a technical metric (tokens/s) to a human stakes argument without feeling like marketing. It's also strategically smart: it positions inference speed as a *safety* and *healthcare* concern, not just a developer convenience. That's how you justify premium pricing to enterprise buyers.

## Key Themes

- **#concept** — Model-system codesign as methodology: the model and inference stack are co-designed, not layered
- **#tool** — MiMo-V2.5-Pro: Xiaomi's 1T-parameter MoE flagship model
- **#tool** — TileRT: ultra-low-latency inference runtime with persistent kernels and warp specialization
- **#tool** — DFlash: block-level masked parallel prediction for speculative decoding
- **#concept** — FP4 (MXFP4) quantization of MoE Experts only, preserving original precision for attention/routing
- **#pattern** — Speed → Intelligence: using excess inference speed as a budget for parallel reasoning chains

## Demos

- **Snake game** — built from scratch in ~10 seconds
- **MacOS interface** — recreated in ~1 minute

These aren't just parlor tricks. The Snake demo in particular demonstrates the coding agent thesis: at 1000 tps, the feedback loop between "write code" and "see it run" collapses to near-zero. That's the qualitative shift from "AI as code generator" to "AI as real-time programming partner."

## Open Source

The **MiMo-V2.5-Pro-FP4-DFlash** checkpoint is on HuggingFace at [huggingface.co/XiaomiMiMo/MiMo-V2.5-Pro-FP4-DFlash](https://huggingface.co/XiaomiMiMo/MiMo-V2.5-Pro-FP4-DFlash) — FP4 quantized weights plus DFlash model parameters. UltraSpeed support for the base MiMo-V2.5 model is "on the way."

The DFlash paper is at [arxiv.org/abs/2602.06036](https://arxiv.org/abs/2602.06036). The OCP MXFP4 spec: [opencompute.org](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf).

## Critical Analysis

**The commodity GPU claim is the real differentiator.** Cerebras (wafer-scale) and Groq (SRAM-only) both achieved extreme speeds earlier, but on custom silicon that's expensive and scarce. MiMo-TileRT doing 1000 tps on standard 8-GPU nodes is a fundamentally different category — it's reproducible. The open-source release of the FP4-DFlash checkpoint on HuggingFace reinforces this: they want people to run it.

The same codesign methodology, pointed at a moving target, is [[FreeToken — Edge-Native MoE Serving]]: instead of tuning one engine to one fixed 8-GPU node, it continuously remaps expert residency and CPU–GPU execution onto whatever heterogeneous balance an individual laptop or workstation exposes. Where MiMo/TileRT optimized a known rig, FreeToken adapts to *your* rig — the edge-native branch of the same idea.

**The selective FP4 strategy is clever and honest.** Quantizing only the MoE Experts (the bulk of parameters) while keeping attention and routing at full precision is a pragmatic tradeoff. They show benchmarks confirming "essentially on par" with FP8 — but notably, they don't claim lossless. The honesty about degradation from "naive FP4 across the entire model" is refreshing compared to quantization papers that bury the caveats.

**DFlash acceptance rates are strong but uneven.** Coding (6.30/8) and Math/Reasoning (5.56/8) are impressive. Agent (4.29/8) is decent. But they explicitly call out that general conversation acceptance rates "are not yet high." This is the right kind of candor — it tells you where this technique works and where it doesn't. The block size of 8 is a deliberate engineering choice: small enough that the verification pass is cheap, large enough that 6–7 accepted tokens per round is a real win.

**The TileRT execution model is the hidden hero.** The article gives equal billing to MiMo and TileRT, but reading between the lines, TileRT's persistent-kernel architecture is doing the heavy lifting on the systems side. "Persistent Engine Kernel" and "Warp Specialization" aren't marketing terms — they're describing fundamental changes to how GPU work is scheduled. This is the kind of work that NVIDIA's CUDA team would be interested in.

**The 3× price / 10× speed framing is a savvy go-to-market.** It's not cheaper per token — it's more expensive, but you get results 10× faster. This positions UltraSpeed as a premium tier for latency-sensitive workloads (coding agents, real-time decision systems) rather than a cost-reduction play. The application-based access with enterprise prioritization confirms they see this as a business product, not a mass-market API.

**What's missing:** No latency numbers for time-to-first-token. 1000 tps decode speed is impressive, but for interactive use, the prefill latency (processing the prompt before generation starts) matters just as much. Also no numbers on batch throughput — can they sustain 1000 tps with multiple concurrent users, or is this single-stream? And the two-week trial window (June 9–23) with application gating suggests this is more demo than product right now.

**The speed-to-intelligence argument deserves scrutiny.** Running Best-of-N in parallel is a real capability gain, but it's not "intelligence" — it's compute. The claim that "speed transmutes into intelligence" conflates search over reasoning paths with actual reasoning capability. Still, for many practical purposes (coding, math verification), search IS the difference between a right answer and a wrong one.

## Cross-References

- [[KV Cache Locality]] — prefix-aware routing for GPU inference efficiency; the complementary problem to decode speed
- [[Muse Spark]] — Meta's 10× compute efficiency claim on proprietary models; compare the approaches
- [[Smart Models Dumb Pipes]] — the end-to-end principle: MiMo does the smart part, TileRT is the dumb (fast) pipe done right
- [[How Far Behind Are Open Models]] — MiMo-V2.5 sits in the open model tier; this speed breakthrough partially closes the practical gap
- [[Writing Code vs. Shipping Code]] — the coding agent productivity claim here (1000 tps unlocks agents) intersects directly with the Demirer et al. finding that AI productivity gains attenuate at release
- [[2025 in LLMs]] — Simon Willison's annual survey; MiMo is one of the new entrants worth tracking

---
*Sources: [[summary/mimo-tilert-1000tps]], [MiMo-V2.5-Pro-FP4-DFlash on HuggingFace](https://huggingface.co/XiaomiMiMo/MiMo-V2.5-Pro-FP4-DFlash), [DFlash paper (arXiv:2602.06036)](https://arxiv.org/abs/2602.06036), [TileRT technical details](https://www.tilert.ai/blog/breaking-1000-tps.html)*
*Last updated: 2026-06-09*
