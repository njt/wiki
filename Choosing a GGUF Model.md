# Choosing a GGUF Model

A practical field guide to GGUF quantization formats: the three families (legacy Q_0, K-quants, I-quants), how they differ in design and trade-offs, and which to pick for your hardware and quality requirements. Benjamin Marie's piece is the reference the local-inference community keeps pointing people at — and for good reason. It's the clearest single-source taxonomy of the alphabet soup that confronts anyone downloading models from Hugging Face.

---

## Key Quotes

> "K-quants generally match or beat legacy formats in throughput because you move fewer bytes for the same quality."

This is the single sentence that justifies the transition from legacy formats. K-quants aren't just better quality — they're *faster* at equivalent quality because the two-level super-block scheme cuts metadata overhead enough to more than pay for the extra dequantization work. On modern hardware, there is no reason to use Q4_0 over Q4_K_M unless you're on a platform where the K-quant kernels haven't been ported.

> Q4_K_M is "a widely useful default for 4-bit deployments"

Marie puts his thumb on the scale here, and he's transparent about why: of the models he personally uploads, Q4_K_M is always the most downloaded variant. This is a revealed-preference argument dressed as a recommendation. It's also correct — Q4_K_M hits the knee of the quality-per-byte curve for most models above 8B parameters. The `_M` suffix matters: it spends 5–6 bits on attention value projections and final layers, which disproportionately affect output quality.

> The I-quant "advantage is real, but it is not uniform across the whole family."

This is the most important caveat in the article and the one most people skip. I-quants are not a monotonic upgrade over K-quants. IQ2_XXS at ~2.1 bpw is a different beast from IQ4_XS at ~4.5 bpw. The importance-matrix approach amplifies whatever biases exist in the calibration data — a bad imatrix produces a bad I-quant, period. Marie is diplomatic about this; the blunt version is that a poorly-produced IQ4_XS is worse than a well-produced Q4_K_M, and you can't tell which you're getting from the filename alone.

> "Some quant publishers drop IQ4_NL as 'redundant' unless you've tested a specific CPU/hardware path where it wins."

IQ4_NL occupies an awkward position: it's a non-linear 4-bit format with 32-weight blocks designed for CPU-friendliness, but benchmarks place it within noise of IQ4_XS for most GPU use cases. It exists because the 256-weight super-block design of standard I-quants doesn't work for every tensor shape, not because there's a clear user-facing reason to prefer it. The community's instinct to treat it as a compatibility shim rather than a first-class option is correct.

---

## Key Themes

- `#concept` **Blockwise quantization**: The core idea behind GGUF — split weight matrices into fixed-size blocks, quantize each block independently with local parameters, reconstruct at inference time. The three design knobs are bits/weight, block size, and dequantization rule.
- `#concept` **Two-level quantization (K-quants)**: Super-blocks (256 weights) with global scale/offset containing sub-blocks (32 weights) with local scale/zero-point. Double-quantization of per-group scales. A piecewise-affine approximation that captures both local and global weight distribution shape.
- `#concept` **Importance-matrix quantization (I-quants)**: Reconstruction guided by an importance matrix — weights that matter more for the loss get more bits de facto. The quality ceiling is higher than K-quants but the floor is lower.
- `#tool` **GGUF**: The file format that serializes quantized weights alongside metadata, tokenizer, and model architecture config. The de facto standard for local inference via llama.cpp and derivatives.
- `#pattern` **Mixed-precision quantization**: Not all layers are equally sensitive. Store embeddings, final output projections, and KV heads at higher precision; quantize the bulk of MLP and attention weights aggressively. Q4_K_M and Q5_K_M bake this pattern into their `_M` suffix design.
- `#pattern` **The imatrix dependency**: I-quant quality is gated on calibration data quality. A quant produced with a mismatched imatrix is worse than a K-quant at the same bitrate. This is a supply-chain problem — downloaders can't verify imatrix provenance from the filename.

---

## Critical Analysis

**The taxonomy is excellent; the empirics are paywalled.** The article's real value is in the clear, technically-precise description of *how* each format family works — the block structure, the dequantization rule, the metadata overhead trade-offs. That's the hard part and Marie nails it. But the "Accuracy, Size, and Speed" section with actual numbers is behind the Substack paywall, which means this article functions as a conceptual map, not an empirical reference. You'll need to bring your own perplexity benchmarks.

**Marie has a conflict of interest, and he's transparent about it.** He uploads GGUF models to Hugging Face and his download stats tell him Q4_K_M wins. That's useful signal, but it's also selection bias: the people downloading his quants are self-selected to prefer the format he recommends. The people who need IQ2_XXS to cram a 70B model onto a 24GB card might be downloading someone else's quants.

**The S/M/L suffix taxonomy is under-explained.** Marie uses Q4_K as his running example for the mix levels, which makes sense — it's the most popular. But the article doesn't explain how the mix strategy changes at other bitrates. Q2_K_L raises different questions than Q5_K_S. The suffix system is a coarse knob on a continuous trade-off, and the article doesn't give you a heuristic for when to reach for an off-nominal variant.

**This is a snapshot of a moving target.** The article was published October 2025 and the llama.cpp quantization landscape evolves quickly. Since publication, FP4 hardware support has become a practical consideration (see [[Local Models in Mid-2026]]), and the IQ family has expanded. Marie's core framework — understand the dequantization rule, match it to your quality/speed/size priorities — is durable. The specific format rankings will drift.

**Practical takeaway:** For any model ≥8B parameters on modern hardware, start with Q4_K_M. If you're VRAM-constrained, try IQ4_XS with a reputable publisher's imatrix. If you're on CPU, benchmark IQ4_NL against Q4_K_M on your specific hardware before committing. For extreme compression (fitting 70B+ on consumer GPUs), the IQ2/IQ3 family is your only option — but calibrate your expectations accordingly. And if someone tries to sell you on Q4_0 in 2026, ask what platform they're targeting that doesn't have K-quant kernels.

---

## Cross-References

- [[Local and Open Source Inference]] — Hub page for running models on your own hardware. GGUF is the de facto format for this ecosystem.
- [[Local Models in Mid-2026]] — Coles' survey of the local model landscape, including the role of 4-bit quantization in closing the open/closed gap and the DRAM price irony.
- [[Inference Cost Napkin Math]] — Why memory bandwidth is the binding constraint: the arithmetic that makes quantization matter in the first place.
- [[AirLLM]] — The alternative to quantization: stream one layer at a time from disk. Complementary approach for when even 4-bit won't fit.
- [[Datacenter GPU in a Gaming PC]] — Practical local hardware setup where GGUF format selection is part of the daily workflow.
- [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] — A concrete two-GPU config where quant choice (Q8 in this case) is the difference between fitting and not fitting.
- [[LocalAI]] — Local inference platform that uses GGUF as its primary model format.
- [[Well-Read Students Learn Better]] — The 2019 paper that showed pre-training compact models beats elaborate compression. The counterpoint: sometimes you want the big model, quantized.

---
*Source: [[raw/choosing-gguf-model-k-quants-i]]*
*Last updated: 2026-07-05*
