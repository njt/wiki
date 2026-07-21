# Theoretical LLM Inference Bottlenecks

Freddie Spirit's definitive first-principles derivation of every bottleneck in LLM inference, structured as a hierarchy of ceilings from the roofline model up. If you only read one technical article on why LLM inference is slow and what can be done about it, make it this one.

---

## The Big Picture

Spirit builds from a single accounting identity — execution time is bounded by max(compute, memory, communication) — and derives the entire inference optimization landscape as a series of consequences. The roofline model is the master key: arithmetic intensity I = FLOPs / bytes_moved, and your achievable performance is min(peak_FLOPS, I × bandwidth). The H100's ridge point is ~295 FLOPs/byte. Everything else follows.

The central asymmetry: **prefill is compute-bound** (I ≈ 2048 at S=2048, far above the ridge), while **decode at batch-1 is catastrophically bandwidth-bound** (I ≈ 1, two-and-a-half orders of magnitude below the ridge). This single fact explains why inference optimization is overwhelmingly about memory movement, not FLOPs.

## Key Quotes

> "At batch size 1, decode intensity ≈ 1 FLOP/byte. That's two-and-a-half orders of magnitude below the ridge point."

The number that structures the entire field. If you're not thinking about bandwidth first, you're thinking about the wrong thing.

> "You could double the TFLOPS and gain exactly nothing at batch 1."

The cleanest possible refutation of "just wait for faster GPUs." Compute throughput is irrelevant to the single-stream experience. This is why [[MiMo-V2.5-Pro-UltraSpeed]] and TileRT's 1000+ tok/s achievement required model-system codesign attacking the bandwidth term — not just faster silicon.

> "Token t+1's computation depends on token t's output; the forward passes cannot be parallelized across the sequence dimension at generation time. This is a dependency-chain limit, not a resource limit."

The distinction that most inference discussions miss. Latency and throughput are different problems with different ceilings. Batching fixes throughput but does nothing for single-stream speed. This taxonomy — batching: throughput only; quantization: both; speculative decoding: latency specifically — is worth tattooing on your forearm.

> "Push P higher and you hit Amdahl's law: as the memory term → 0, tokens/sec asymptotes to 1 / (comm_floor) ≈ 900 tok/s regardless of GPU count."

Tensor parallelism has a hard communication floor set by n_layers × all-reduce latency. You can't parallelize your way past it — and crossing node boundaries (InfiniBand's 2-10× worse latency) makes it dramatically worse. This is why single-node inference is the default and distributed inference is pain.

> "Speculative decoding spends spare compute to buy latency; in throughput-saturated serving, that spare compute doesn't exist, and speculation can reduce aggregate throughput."

Speculative decoding is a latency tool, not a throughput tool — the taxonomy again. In compute-bound regimes (large batch), the "spare" compute it depends on doesn't exist. Misusing it is surprisingly common.

## The Hierarchy of Ceilings

Spirit assembles the complete picture as two hierarchies:

**Single-stream decode** (tightest first):
1. Sequential dependency — forbids parallelizing across positions
2. Communication latency floor under parallelism — ~1 ms/token, immune to more hardware
3. Memory bandwidth — attackable by quantization, GQA, sharding
4. Soft overheads — kernel launch, bandwidth utilization, attention kernels, KV cache fragmentation

**Aggregate throughput**:
1. KV cache capacity caps batch size
2. Compute saturation caps FLOPs
3. Scheduling efficiency determines how close you run to those caps

## Back-of-Envelope Exercises

Spirit includes worked examples that double as a self-test:

- **70B fp16 on H100, batch-1**: 140 GB / 3.35 TB/s → ~24 tok/s ceiling. Real: ~15-19 tok/s.
- **8B fp16 on H100, batch-1**: 16 GB / 3.35 TB/s → ~210 tok/s ceiling. Real: ~150-180 tok/s.
- **405B fp8 on 8×H100, batch-1**: 405/8/3.35 + (126×2×7μs) ≈ 16.9 ms → ~59 tok/s ceiling.

The pattern: real systems hit 60-80% of the theoretical bound. The gap is the soft bottlenecks — kernel launch overhead, bandwidth utilization inefficiency, KV cache fragmentation. Collectively worth 2-5×, which is why systems engineering matters as much as algorithms.

## Critical Analysis

**What makes this exceptional:** Spirit teaches the *method*, not just the answers. Every section follows the same structure: state the bound, derive why it binds, show what it doesn't depend on (often the most illuminating part), then derive the ceiling and check against reality. The closing challenge — "derive the batch-1 ceiling for any model-hardware pair unprompted" — is earned. You can.

**What's conspicuously absent:** The entire analysis assumes dense transformer architectures. MoE models (like [[Cohere North Mini Code]] with 30B total / 3B active parameters) change the roofline math because active parameters per token are much smaller than total parameters — the bandwidth term shrinks by the expert activation ratio. Spirit doesn't address this, and it's the most important omission for the MiMo/Xiaomi/TileRT generation of models hitting 1000+ tps. Similarly, the analysis assumes H100-era hardware; [[NVIDIA B300 vs H200 GPU Analysis|the B300's 8 TB/s bandwidth]] shifts the ridge point significantly.

**The pedagogical structure is the real contribution:** This isn't novel research — it's a synthesis of well-known results (roofline, Amdahl, FlashAttention's roofline argument, PagedAttention's fragmentation argument). But nobody had assembled them into a single derivable-from-first-principles hierarchy. Spirit turned inference optimization from a bag of tricks into a deductive system. That's genuine intellectual infrastructure.

**What to read next:** [[Inference Cost Napkin Math]] for the economic consequences of the same bandwidth bottleneck. [[KV Cache Locality]] for the practical implication of the KV cache capacity bound. [[MiMo-V2.5-Pro-UltraSpeed]] for the state of the art in attacking these ceilings. [[The Tokens You Can't Wait For]] for text diffusion as an architectural escape hatch — parallel generation makes decode compute-bound even at batch size one, bypassing the dependency-chain limit that this article derives. [[GPU-Free AI Datacenters]] for the radical thesis that the entire bottleneck hierarchy is downstream of computational assumptions we could abandon.

---
*Sources: [[summary/theoretical-llm-inference-bottlenecks]]*
*Last updated: 2026-07-05*
