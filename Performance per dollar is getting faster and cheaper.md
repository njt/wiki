# Performance per dollar is getting faster and cheaper

Wafer.ai's field report on running GLM-5.2 inference on AMD MI355X GPUs: 80% of NVIDIA B200 throughput at less than half the cost, achieved with framework-level fixes rather than custom kernels. The real story is that the CUDA moat is eroding from the top down — not because AMD's software stack is great, but because the remaining gaps are small enough that a few `#ifdef` guards and config tweaks close them.

---

## Key Quotes

> "Only 80% of the performance measured on a B200, despite being over 2x cheaper."

This is the headline number and Ian Ye puts it up front. He's not claiming AMD beats NVIDIA on raw throughput — he's claiming it wins on *performance per dollar*, which is the metric that actually matters for inference providers who pay their own hardware bills. The ratio works out to roughly 1.6x better price-performance. For a cost-center workload like inference, that's not a rounding error; it's a business model.

> "Close to a 3x gain in single stream throughput" from fixing the MTP head quantization mismatch.

The fix itself was trivial — copy layer-78 entries into Quark's un-quantized list under the name sglang expects. But the 3x gain reveals how much performance is hiding behind small software incompatibilities. This is the real state of AMD inference in mid-2026: the hardware is capable, but you'll spend days hunting down prefix mismatches and missing ROCm guards that NVIDIA users never think about.

> "SOTA on AMD is becoming more a matter of support, not software."

Ye's thesis, stated plainly. The killer line is the contrast with prior work — their Qwen3.5 397B deployment required writing custom kernels from scratch. GLM-5.2 on MI355X required zero custom kernels. The gap between "impossible without GPU expertise" and "annoying but doable" collapsed in under two months. That's the trend that should worry NVIDIA.

> "The CUDA moat is eroding in real time."

Not subtle, but earned. When a small team can get a frontier MoE model running competitively on AMD hardware with a few `#ifdef USE_ROCM` guards and a kernel selection tuning pass, the moat isn't a moat anymore — it's a speed bump. The remaining friction is documentation gaps and missing tuned configs, not architectural lock-in.

## What They Actually Did

The team quantized GLM-5.2 from bf16 to MXFP4 using AMD's Quark tool. Across three evals (GSM8K, GPQA-Diamond, tau2 macro), the MXFP4 version was essentially lossless versus z-ai's official FP8 — single-digit basis-point differences, indistinguishable from noise.

They chose **sglang** over vLLM (no working MXFP4 + GlmMoeDsa path) and ATOM (degraded at long context). Two bugs blocked speculative decode: an MTP head prefix mismatch between Quark's module naming and sglang's, and a CUDA-only header include in a fused metadata kernel. Both were one-line fixes.

The prefill optimization story is instructive: TP8 gave 1,461 tok/s/node. Switching to TP4×DP2 boosted this to 1,944. Then they discovered GLM-5.2's fp4 MoE was running on a slow FlyDSL heuristic fallback because aiter only shipped tuned configs for fp8. A manual kernel-selection tuning pass took them to 2,626 tok/s/node at 2.4 RPS.

Final numbers: 2,626 tok/s/node aggregate, 213 tok/s single-stream decode, ≤5s TTFT. On hardware that costs less than half what the equivalent NVIDIA node costs.

## Key Themes

- `#comparison` **AMD vs NVIDIA for inference** — the price-performance gap now favors AMD, and the software gap is shrinking fast. The remaining friction is support maturity, not fundamental architecture.
- `#tool` **sglang** — wins the inference framework bake-off for AMD deployments. vLLM's MXFP4 path was broken; ATOM degraded at long context. Sglang had the least friction for native quantization support, which is becoming the deciding factor.
- `#tool` **AMD Quark** — AMD's quantization toolkit produced a near-lossless MXFP4 version of GLM-5.2. The tool works; the sharp edges are in module naming conventions and kernel config coverage.
- `#concept` **MXFP4 quantization** — 4-bit floating-point quantization that preserves model quality across evals. The third generation of quantization formats after GGUF's K-quants and I-quants, and the one that matters for datacenter deployment.
- `#pattern` **Speculative decode as inference multiplier** — Multi-Token Prediction delivered a 3x single-stream throughput gain once the software bugs were fixed. This is becoming table stakes for production inference, not an optional optimization.
- `#concept` **CUDA moat erosion** — the structural trend: each generation of AMD hardware closes more of the software gap, and the remaining issues are increasingly trivial (missing `#ifdef` guards, untuned kernel configs) rather than architectural.

## Critical Analysis

Ye's framing is honest in a way most hardware comparison pieces aren't. He doesn't pretend AMD is plug-and-play or that the software experience matches NVIDIA's. The article is essentially a list of things that broke and how they fixed them. That candor makes the conclusion more credible, not less — when someone shows you all the warts and still concludes the economics work, you pay attention.

But there's a tension he doesn't fully explore. The two bugs they hit (MTP head prefix mismatch, missing ROCm guard) are exactly the kind of papercuts that add up to real engineering cost at scale. A one-line `#ifdef` takes an hour to diagnose and three days to upstream. The kernel selection tuning pass required understanding aiter internals that most teams don't have. The article's implicit argument is "any competent team can do this," but "competent" here means "has someone who understands both the model architecture and the inference framework internals." That's a small pool.

The bigger question: does this economics argument hold at scale, or is it a single-node demo? Managing a fleet of MI355X nodes for production inference means dealing with AMD's driver stack, ROCm version compatibility, and a much smaller community for troubleshooting. The per-node savings are real, but the operational overhead might eat them. Ye doesn't address this because his piece is a technical field report, not a TCO analysis — but anyone making purchasing decisions needs that second number.

That said, the trajectory is unmistakable. Two months earlier, the same team needed custom kernels for Qwen3.5 397B on MI355X. Now they don't. If the trend holds, the next generation won't even need the `#ifdef` guards. NVIDIA's moat isn't just eroding — it's eroding at an accelerating rate, and the only question is when, not whether, AMD becomes the default choice for inference providers who care about margins.

The [[GLM-5.2 Is the Step Change for Open Agents]] context matters here too. GLM-5.2 is MIT-licensed and competitive with Opus 4.8 as a coding agent. If the best available open-weight coding model runs cost-effectively on AMD hardware, the inference economics shift from "NVIDIA for training, maybe AMD for inference" to "AMD for everything except training." That's a much bigger market than most people realize.

---

*Sources: [[raw/glm52-amd]]*
*Last updated: 2026-07-05*
*Cross-references: [[GLM-5.2 Is the Step Change for Open Agents]], [[Inference Cost Napkin Math]], [[Local Models in Mid-2026]], [[Choosing a GGUF Model]], [[MiMo-V2.5-Pro-UltraSpeed]], [[KV Cache Locality]]*
