---
url: https://kaitchup.substack.com/p/choosing-a-gguf-model-k-quants-i
date_fetched: 2026-07-05
backfilled: true
---

# Choosing a GGUF Model: K-Quants, IQ Variants, and Legacy Formats

### Reviewing the differences between each type and their impact on accuracy, throughput, and memory.

For local LLM inference, the GGUF format, introduced by llama.cpp and popularized by frontends like Ollama, is by far the most common choice.

Each major LLM release is quickly followed by a wave of community GGUF conversions on the Hugging Face Hub. Prominent curators include Unsloth and Bartowski (also: TheBloke remains widely used), among many others. Repos often provide dozens of variants per model tuned for different memory/quality trade-offs.

For instance, Unsloth released 25 GGUF versions of Qwen3 8B and 26 versions for DeepSeek-V3.1-Terminus.

That’s a lot of choice, but beyond filename and size, there’s rarely a clear guide to accuracy, speed, or trade-offs for each format. New variants land regularly, so I wrote this guide to demystify the main GGUF-serializable formats across architectures: how they work, why their accuracy/size/throughput differ, and when to pick each one. (This guide doesn’t cover converting your own models; I’ve written about that separately.)

If you are looking for “How to Run GGUF Models,” check this article.

## What Is GGUF?

I introduced GGUF in this article:


TL;DRMost GGUF weight formats are blockwise.

A matrix is split into fixed-size blocks, each block is represented with compact integer parameters, and a small set of per-block parameters reconstructs approximate floating weights at inference.

The design space is defined by three choices:


The number of bits used for the parameters

The block size

The dequantization rule (linear scale and zero-point, multi-scale hierarchies, or non-linear/LUT-assisted schemes)

The more expressive the dequantization rule, the lower the error you can achieve for the same number of bits, at some decode cost.


In the next sections, “bits/weight” refers to the effective average once overheads like block scales are included. Values are approximate and vary a little by implementation and tensor shape, but they are useful for thinking about trade-offs.

## Legacy Formats: Q_0 and Q_1

The legacy family of GGUF formats, Q4_0, Q4_1, Q5_0, Q5_1, Q8_0, implements classic per-block linear quantization. A block stores n-bit weight codes and either one scale (the “_0” variants, symmetric) or one scale plus one offset/zero-point (the “_1” variants, asymmetric). Dequantization is a single affine transform per block.

These formats are simple to decode and therefore fast. Their weakness is representational: one affine map per block cannot model skewed or heavy-tailed weight distributions as well as newer schemes.

At 8-bit, the difference is negligible, and Q8_0 is effectively near-lossless for most LLMs. That’s why we can still see a lot of Q8_0 models being published on the HF Hub. At 5- and especially 4-bit, legacy formats leave measurable accuracy on the table compared with modern alternatives. They remain relevant for maximum simplicity and compatibility, and on some older devices, their very cheap decoding can still be a speed win.

A concise way to think about the legacy set is that Q8_0 is a safe INT8 baseline, Q5_0/1 are decent mid-range choices if you must stick to legacy, and Q4_0/1 are largely superseded by K- and I-quants for quality per bit.

## K-quants: Modern Default for 3–6 Bits

K-quants (Q2_K, Q3_K, Q4_K, Q5_K, Q6_K, and their mixed variants like _S, _M, _L) introduce structure beyond a single affine per block. We saw how to make these models here:

The most common pattern is a two-level scheme: small blocks with their own scale and zero-point grouped into a super-block with an additional scale/offset. In practice, this behaves like a piecewise-affine approximation that captures both local and global variation with little overhead.

This is an asymmetric quantization scheme (most variants map negatives and positives to different ranges), with the exceptions of Q3_K and Q5_K which are symmetric. They quantize weights in fixed-size groups (32-weight blocks packed into 256-weight “super-blocks”) and apply double-quantization to the per-group scales, first computing a scale for each group, then quantizing those scales again, reducing metadata overhead and improving quality-per-bit compared to legacy formats.

The result is lower error at the same storage. For example, a typical Q4_K lands around the mid-4s bits/weight—slightly above Q4_0/1 once you count its extra parameters, but it achieves distinctly better fidelity. Q5_K and Q6_K cluster close to the original model in perplexity while remaining far smaller than FP16.

Decoding remains lightweight. The extra parameters are compact, and arithmetic is still simple integer unpacking plus a handful of multiplies and adds. On modern CPUs and GPUs, K-quants generally match or beat legacy formats in throughput because you move fewer bytes for the same quality.

The suffixes encode “mix levels” across tensors. Examples for Q4_K:

- Q4_K_S (small): Keeps almost everything at 4-bit 
- Q4_K_M (medium): Selectively raises precision for more sensitive tensors (for example, attention value projections or final layers) using 5–6 bits 
- Q4_K_L (larger): Even more relaxed than Q4_K_M. 

The effective bits/weight rise accordingly, buying back quality where it matters. In practice, Q4_K_M is a widely useful default for 4-bit deployments (Q4_K is also OK for large models). Q5_K_M is a high-quality setting that is close to imperceptible degradation for many tasks. Q6_K is for cases where you want “almost lossless” behavior and still want memory savings.

Keep in mind that for most models, you won’t see much difference in quality between S, M, and L variants, unless you are dealing with small models (let’s say <8B models).

### Q4_K_M Is the Most Popular

When I propose GGUF models, the Q4_K_M variant is always the most downloaded. It’s not very surprising. It has a low memory footprint and is often as accurate as the original model.

### The Special Case of TQ1_0

For very large LLMs, like DeepSeek models, you may also find a TQ1_0 version.

TQ1_0 encodes weights that are ternary (values in {−1, 0, +1}) using a compact packing scheme. It lands around ~1.6–1.7 bits/weight depending on packing details.

## I-Quants: Pushing Quality at Lower Precision

I-quants such as `IQ2_XXS`, `IQ2_XS`, `IQ2_S`, `IQ3_XXS`, `IQ3_XS`, `IQ3_S`, `IQ3_M`, `IQ4_XS`, and `IQ4_NL` are GGUF quantization formats aimed at preserving as much quality as possible at low bitrates. The IQ family is defined around importance-matrix-based reconstruction: weights are recovered using a super-block scale and an importance matrix. Most listed IQ formats use 256-weight super-blocks, while `IQ4_NL` is the main exception and uses 32-weight blocks with a non-linear mapping.

Compared with K-quants, the main appeal of I-quants is usually quality per byte, especially when a good imatrix is available. That advantage is real, but it is not uniform across the whole family.

That means the `XXS` / `XS` / `S` / `M` suffixes are best understood as different operating points on the size-quality spectrum, not as a universal ranking. More aggressive variants such as `XXS` minimize size, while larger variants generally spend more bits per weight to retain more accuracy.

`IQ4_NL` is a special case within the family. It was introduced as a 4-bit non-linear quantization type with 32-weight blocks for cases where 256-weight k- and i-quants are not available.

## IQ4_XS vs Q4_K_M

I get this question very often: Should I use IQ4_XS over Q4_K_M? So, let’s focus on these 4-bit variants.

IQ4_XS and Q4_K_M are both “4-bit class” GGUF quantizations, but they trade off size, speed, and robustness differently: Q4_K_M is the reliable default (slightly larger, generally predictable quality/perf), while IQ4_XS is compresses more aggressively (lower effective bits/weight), which can help you fit larger models/contexts and sometimes improve token generation speed, at the cost of being more sensitive to how the quant was produced (imatrix quality; see below) and to your hardware/kernel mix. In llama.cpp’s published Llama-3.1-8B numbers, IQ4_XS is ~**4.46 bpw / 4.17 GiB** vs Q4_K_M at ~**4.89 bpw / 4.58 GiB**, with **IQ4_XS a bit faster for generation but a bit slower on prompt processing.**

## IQ4_NL vs IQ4_XS

IQ4_NL and IQ4_XS are both llama.cpp “I-quant” GGUF formats aimed at strong quality at ~4-bit, but they optimize different things: IQ4_XS is the more aggressive/compressed option (~4.25 bpw) and reconstructs weights using an importance-matrix–style scheme, which can buy you extra headroom for VRAM/context at the cost of being a bit more sensitive to how the quant was produced. As mentioned above, IQ4_NL is a less compressed non-linear variant (~4.5 bpw) with a different dequantization rule and smaller-block design that’s often described as targeting CPU friendliness/speed while keeping the non-linear benefits. In practice, many community benchmarks report IQ4_NL is very close to IQ4_XS (sometimes within noise), which is why some quant publishers drop IQ4_NL as “redundant” unless you’ve tested a specific CPU/hardware path where it wins.

## What Else Can GGUF Serialize?

Beyond the families above, GGUF also supports unquantized tensors (FP32/FP16/BF16) for layers you choose to leave “full-precision” and hybrid models where some matrices use different formats.

You will often encounter mixed-precision checkpoints where embeddings, final output layers, or KV projections are stored at higher precision while the bulk of MLP and attention weights use K- or I-quants.
