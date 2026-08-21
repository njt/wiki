---
url: https://github.com/ARahim3/mlx-dspark
title: "mlx-dspark"
author: erahim3 (ARahim3)
date_fetched: 2026-08-21
published: 2026 (repo; v0.14.0)
tags: [tool, inference, speculative-decoding, mlx, apple-silicon]
---

**mlx-dspark** is a speculative-decoding inference library for Apple Silicon built on MLX (`mlx-lm` / `mlx-vlm`). It runs the DSpark and DFlash drafter families — semi-autoregressive and block-diffusion drafters trained for target models like Bonsai 27B and Muse-Glimmer — plus a drafter-free n-gram "lookup" mode, and wraps them in a hardware-aware auto-calibration layer that tunes the draft length to whatever Mac it's running on.

The core guarantee is losslessness: the target model verifies every proposed token, so greedy output equals plain decoding (up to floating-point ties) and temperature > 0 is an exact sample from the target via speculative sampling. The engineering edge is Apple-Silicon-specific: multi-token verification leaves the cheap "few rows" path of quantized matmul, so the cost of a wider verify round has a knee that depends on the exact chip, MLX version, and quantization — a property the library measures once per machine and caches.

Three verification modes share one verify loop:
- **DSpark** — an EAGLE-style drafter: a 5-layer backbone with a rank-256 Markov head and a confidence head, cross-attending from the current block over the cached context.
- **DFlash** — a block-diffusion drafter that denoises a whole 16-token block in parallel, reusing the target's embedding table and `lm_head`.
- **Lookup (n-gram)** — no drafter at all: find the latest earlier occurrence of the current suffix and propose the tokens that followed it, verified exactly like any draft.

Target support spans dense (`qwen3`, `gemma4`), hybrid linear-attention (`qwen3_5` gated-DeltaNet, `nemotron_h` Mamba-2), and VLM-without-a-capture-hook (`muse_glimmer`) families. Hybrid recurrent state can't be trimmed like a KV cache, so mlx-dspark records each verify round's recurrence inputs and rebuilds the linear-attention state bit-exactly at the accept point on a partial rejection.

Beyond single-stream generation it ships a continuous-batching server (`--max-batch`), prefix caching with divergence "anchors" and mid-prefill "rungs," reduced draft vocabularies, MoE-aware routing, and an Anthropic Messages API translation for use with Claude Code. MIT-licensed.
