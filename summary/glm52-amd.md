---
url: https://www.wafer.ai/blog/glm52-amd
title: "Performance per dollar is getting faster and cheaper"
author: Ian Ye
date_fetched: 2026-07-05
date_published: 2026-07-03
site: Wafer Blog
---

The article argues that AMD's MI355X GPUs offer a superior price-to-performance ratio for AI inference compared to NVIDIA's Blackwell (B300/B200) lineup, despite requiring more engineering effort to unlock that performance.

## Key Performance Claims

- **Aggregate throughput (20k in / 1k out, 60% cache hit):** 2,626 tok/s/node at 2.4 RPS with ≤5s TTFT — "only 80% of the performance measured on a B200, despite being over 2x cheaper."
- **Single-stream decode:** 213 tok/s on GLM-5.2 (10k input / 1.5k output tokens) following Artificial Analysis standards.
- AMD GPUs are "around 2.75x cheaper per GPU on average (MI355X vs B300)" with comparable specs.

## How They Did It

### Quantization
The team quantized GLM-5.2 from base bf16 to **MXFP4** using AMD's Quark tool. Compared to z-ai's official FP8 quantization, the MXFP4 version was essentially lossless across three evaluations:

| Eval | FP8 baseline | MXFP4 | Δ |
|---|---|---|---|
| GSM8K (200q, 5-shot, greedy) | 0.965 | 0.955 | −0.010 |
| GPQA-Diamond (198q × 2 seeds) | 0.9217 | 0.9026 | −0.019 |
| tau2 macro | 0.819 | 0.834 | +0.015 |

### Inference Framework
Three options were considered — vLLM, ATOM, and sglang. They chose **sglang** because vLLM had no working MXFP4 + GlmMoeDsa path, and ATOM degraded at long context. Sglang offered the least friction for native quantization support.

### Speculative Decode Fixes
Two bugs needed patching to enable Multi-Token Prediction (MTP):

1. **MTP head quantization mismatch**: The MTP shared expert is stored in bf16 but registered under a different module prefix (`model.decoder.*` vs `model.layers.78.mlp.shared_experts.*`). The fix involved copying layer-78 entries into Quark's un-quantized list a second time under the decoder name sglang uses. This yielded "close to a 3x gain in single stream throughput."

2. **CUDA-ism in fused kernel**: Deep spec decode (e.g., 5/1/6 depth) was blocked because a fused multi-step metadata kernel included `<cuda_runtime.h>` without a ROCm guard. Fix: a single `#ifdef USE_ROCM` guard.

Additional config optimizations included `--kv-cache-dtype fp8_e4m3` and `--enable-aiter-allreduce-fusion`.

### Prefill Optimization for Aggregate Throughput
At TP8 (tensor parallelism 8), the MI355X achieved 1,461 tok/s/node for prefill. Switching to **TP4×DP2** boosted this to 1,944 tok/s/node at 2.0 RPS. However, GLM-5.2's fp4 MoE was running on a slow FlyDSL heuristic fallback because aiter only shipped tuned configs for the a8w8/fp8 path. The team "tuned the MoE kernel selection ourselves on GLM's fp4 shapes," hitting 2,626 tok/s/node at 2.4 RPS.

## Why This Matters

The author emphasizes that unlike prior work (e.g., Qwen3.5 397B), this project required **no custom kernel writes** — only framework-level bug fixes. The piece concludes that "SOTA on AMD is becoming more a matter of support, not software" and that "the CUDA moat is eroding in real time."

## Related Articles Referenced

1. **"The Inference Alpha: Maximizing Frontier Models on AMD"** (June 10, 2026) — by Balaji Varadarajan and Wafer Team, covering Kimi 2.5, DeepSeek V3.2, and GLM-5 optimization.
2. **"Achieving Heterogeneous Compute One Kernel at a Time"** (May 19, 2026) — by Ian Ye, detailing custom kernel work for Qwen3.5 397B on MI355X.
