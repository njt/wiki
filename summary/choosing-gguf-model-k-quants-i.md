---
url: https://kaitchup.substack.com/p/choosing-a-gguf-model-k-quants-i
title: "Choosing a GGUF Model: K-Quants, IQ Variants, and Legacy Formats"
author: Benjamin Marie
date_fetched: 2026-07-05
date_published: 2025-10-13
site: "The Kaitchup – AI on a Budget (Substack)"
topics:
  - ai-research-and-models
---

## What Is GGUF?

Benjamin Marie references an earlier article of his for a full introduction. The TL;DR provided in this piece summarizes GGUF weight formats as mostly *blockwise*: a matrix is split into fixed-size blocks, each represented with compact integer parameters, and per-block parameters reconstruct approximate floating weights at inference. Three design choices define the space: bit count per parameter, block size, and the dequantization rule (linear scale/zero-point, multi-scale hierarchies, or non-linear/LUT-assisted schemes). More expressive dequantization rules yield lower error for the same bit cost, at some decode speed trade-off.

The term "bits/weight" throughout refers to the effective average including metadata overheads.

## Legacy Formats: Q_0 and Q_1

This family includes Q4_0, Q4_1, Q5_0, Q5_1, and Q8_0. They implement classic per-block linear quantization. The `_0` variants use symmetric (one scale), while `_1` variants use asymmetric (scale + zero-point/offset). Dequantization is a single affine transform per block.

**Key traits:**

- **Fast decoding** due to simplicity
- **Weakness:** one affine map per block "cannot model skewed or heavy-tailed weight distributions as well as newer schemes"

**Per-format assessment:**

- **Q8_0:** Effectively near-lossless for most LLMs. The difference from modern schemes at 8-bit is negligible. "That's why we can still see a lot of Q8_0 models being published on the HF Hub."
- **Q5_0 / Q5_1:** "decent mid-range choices if you must stick to legacy"
- **Q4_0 / Q4_1:** "largely superseded by K- and I-quants for quality per bit"

Legacy formats remain relevant for "maximum simplicity and compatibility" and can still be a speed win on older devices where cheap decoding matters.

## K-Quants: Modern Default for 3–6 Bits

K-quants (Q2_K, Q3_K, Q4_K, Q5_K, Q6_K, and their mixed variants with `_S`, `_M`, `_L` suffixes) introduce structure beyond a single affine per block.

**How they work:** A two-level scheme — small blocks with their own scale and zero-point are grouped into super-blocks with an additional scale/offset. This "behaves like a piecewise-affine approximation" capturing both local and global variation. Most variants are asymmetric, with Q3_K and Q5_K being symmetric exceptions. Weights are quantized in fixed-size groups (32-weight blocks packed into 256-weight super-blocks), and per-group scales undergo double-quantization, reducing metadata overhead and improving quality-per-bit vs. legacy formats.

**Quality:** Lower error at the same storage. A typical Q4_K lands around mid-4s bits/weight — slightly above Q4_0/1 after counting extra parameters, "but it achieves distinctly better fidelity." Q5_K and Q6_K cluster close to the original model in perplexity while far smaller than FP16.

**Speed:** Decoding remains lightweight. On modern CPUs and GPUs, "K-quants generally match or beat legacy formats in throughput because you move fewer bytes for the same quality."

**Suffix Mix Levels (explained with Q4_K examples):**

| Suffix | Behavior |
|---|---|
| `_S` (Small) | Keeps almost everything at 4-bit |
| `_M` (Medium) | Raises precision for more sensitive tensors (attention value projections, final layers) using 5–6 bits |
| `_L` (Larger) | Even more relaxed than M; buys back more quality |

**Recommendations from the author:**

- **Q4_K_M** is "a widely useful default for 4-bit deployments" (Q4_K is also OK for large models)
- Q5_K_M is "a high-quality setting that is close to imperceptible degradation for many tasks"
- Q6_K is for when you want "almost lossless" behavior with memory savings
- For most models, differences between S, M, L variants are not large *unless* dealing with small models (<8B)

### Q4_K_M Is the Most Popular

The author states that when they propose GGUF models, Q4_K_M is always the most downloaded variant — low memory footprint and often as accurate as the original model.

### The Special Case of TQ1_0

Found for very large LLMs like DeepSeek models. TQ1_0 encodes ternary weights (values in {−1, 0, +1}) using compact packing, landing around ~1.6–1.7 bits/weight.

## I-Quants: Pushing Quality at Lower Precision

I-quants include IQ2_XXS, IQ2_XS, IQ2_S, IQ3_XXS, IQ3_XS, IQ3_S, IQ3_M, IQ4_XS, and IQ4_NL. These are defined around importance-matrix-based reconstruction: weights are recovered using a super-block scale and an importance matrix. Most use 256-weight super-blocks; IQ4_NL is the main exception with 32-weight blocks and non-linear mapping.

**Key point:** The main appeal is quality per byte, especially when a good imatrix is available. However, "that advantage is real, but it is not uniform across the whole family."

The XXS / XS / S / M suffixes are described as "different operating points on the size-quality spectrum, not as a universal ranking." More aggressive variants (XXS) minimize size; larger variants spend more bits per weight to retain accuracy.

**IQ4_NL** is a special 4-bit non-linear quantization type with 32-weight blocks, introduced for cases where 256-weight K- and I-quants are not available.

## IQ4_XS vs Q4_K_M (The FAQ)

This is a direct head-to-head the author says they get asked about often.

| Dimension | Q4_K_M | IQ4_XS |
|---|---|---|
| Role | "reliable default" | More aggressive compression |
| Size | ~4.89 bpw / 4.58 GiB (on Llama-3.1-8B) | ~4.46 bpw / 4.17 GiB (on Llama-3.1-8B) |
| Speed (generation) | Slightly slower | A bit faster |
| Speed (prompt processing) | Faster | A bit slower |
| Sensitivity | More predictable quality/perf across conditions | More sensitive to imatrix quality and hardware/kernel mix |
| Trade-off | Slightly larger, generally predictable | Lower effective bits/weight helps fit larger models/contexts |

## IQ4_NL vs IQ4_XS

Both are I-quants targeting ~4-bit quality, but they optimize different things:

- **IQ4_XS:** More aggressive/compressed (~4.25 bpw), uses importance-matrix-style reconstruction. "can buy you extra headroom for VRAM/context at the cost of being a bit more sensitive to how the quant was produced"
- **IQ4_NL:** Less compressed non-linear variant (~4.5 bpw), uses a different dequantization rule and smaller-block design, described as targeting "CPU friendliness/speed while keeping the non-linear benefits"

The author notes that many community benchmarks report these two as very close (sometimes within noise), which is "why some quant publishers drop IQ4_NL as 'redundant' unless you've tested a specific CPU/hardware path where it wins."

## What Else Can GGUF Serialize?

Beyond quantized formats, GGUF also supports:
- Unquantized tensors (FP32/FP16/BF16) for layers left at "full precision"
- Hybrid models where different matrices use different formats
- Mixed-precision checkpoints where embeddings, final output layers, or KV projections are stored at higher precision while the bulk of MLP and attention weights use K- or I-quants

## Accuracy, Size, and Speed Section

This section is **paywalled** — the article states "This post is for paid subscribers" at that point, so no substantive content is available in the provided text for that section.

## Summary of Recommendations

1. **For compatibility/simplicity on older devices:** Legacy Q8_0 (near-lossless), Q5_0/1 (decent mid-range)
2. **Default 4-bit choice:** Q4_K_M — "always the most downloaded," reliable quality and memory footprint
3. **Higher quality 4-bit:** Q5_K_M — close to imperceptible degradation
4. **Almost lossless with savings:** Q6_K
5. **When VRAM is tight or you need to fit larger models/context:** IQ4_XS (with a good imatrix)
6. **For CPU-friendly non-linear 4-bit:** IQ4_NL (test on your specific hardware)
7. **For very large models where extreme compression is needed:** TQ1_0 (~1.6–1.7 bpw)
8. **Mixed-precision hybrids** for selectively protecting sensitive layers
