# Bonsai 27B

PrismML's Bonsai 27B is the first 27B-class model to run on a phone, built on Qwen 3.6 27B via extreme quantization — ternary weights at 1.71 bpw (5.9 GB) and binary weights at 1.125 bpw (3.9 GB) — with no higher-precision escape hatches anywhere in the network. It's a genuine engineering achievement that opens a new category: frontier-class local inference that fits in a pocket. Apache 2.0.

---

## Key Quotes

> Both variants use low-bit representation across the entire language network — embeddings, attention, MLPs, and the LM head — with no higher-precision escape hatches.

This is the technical headline. Most quantization schemes leave the embedding layer or LM head at higher precision as a safety valve; PrismML burned the boats. That they still retain ~95% (ternary) and ~90% (1-bit) of the full-precision Qwen 3.6 baseline across 15 benchmarks is remarkable. The zero-escape-hatch discipline is what makes the 3.9 GB number real rather than aspirational.

> The marginal cost of a hundred-step loop is zero, and the user's data never leaves the machine.

The economic argument for local agentic inference in one sentence. When each inference step costs cloud credits, agent loops are expensive and architectures get contorted to minimize steps (single-pass, bigger prompts). When they're free, the design space opens up: more iterative refinement, more verification passes, more agentic exploration. The privacy half is equally important — local inference makes privacy the default, not a compliance checkbox.

> A 12 GB iPhone offers roughly 6 GB usable. The 1-bit variant at ~4 GB is the first to pass through with room to work.

This constraint awareness is what separates shipping engineering from research. Phones don't expose full RAM to apps; they never have. PrismML's explicit design target — fit in the *usable* memory budget, not the spec-sheet number — is the kind of product thinking that turns a benchmark result into something someone can actually run.

## Key Themes

- **#concept** **Intelligence density**: The article introduces "intelligence per GB" as a metric, with the 1-bit variant at 0.53/GB — 10× the full-precision baseline. This is a more honest metric than raw benchmark scores for local inference, where the constraint is memory capacity, not compute. It reframes quantization from "degradation management" to "density optimization."

- **#pattern** **Extreme quantization without escape hatches**: Bonsai 27B's refusal to leave any component at higher precision is a design stance, not just an implementation detail. It says: if the math works, commit to it everywhere. This is the opposite of the conservative "quantize the middle layers, leave the head and embeddings at FP16" approach that dominates the field.

- **#tool** **Hybrid local/cloud architecture**: The press release explicitly pitches a model where local handles privacy-sensitive and non-frontier tasks while cloud handles the hardest reasoning. This isn't novel conceptually, but having a phone-sized 27B model makes it architecturally real rather than theoretical. The 1-bit variant handles the routine; the cloud handles the exceptional.

- **#pattern** **Phone as agent host**: A 27B model on a phone that can do tool-calling and agentic work (even at 66.0 on agentic benchmarks) means "personal agent running on the device in your pocket" stops being science fiction. The latency, privacy, and cost profile of a local agent is fundamentally different from a cloud one. See also [[Local and Open Source Inference]].

## Critical Analysis

**The breakthrough is real but the framing is aggressive.** PrismML claims "first 27B-class model to run on a phone," which is technically true in a way that's also carefully bounded — "27B-class" excludes the many smaller models that already run on phones, and "phone" means iPhone 17 Pro, not the median device. The 1-bit variant's 66.0 agentic score (down from 80.0 on the full-precision baseline) is a 17.5% degradation — significant enough that you'd notice it in agentic workflows. This isn't a drop-in replacement for cloud Opus; it's a local model for tasks where "good enough, free, and private" beats "excellent, metered, and remote."

**The zero-escape-hatch decision will age interestingly.** Committing to low-bit precision on embeddings and the LM head is bold, but it means the model has no internal representation at full fidelity. For tasks that depend on subtle semantic distinctions in the embedding space, this could surface as brittleness that benchmark averages don't capture. The [[The Reasoning Trap]] finding — that reasoning-enhancing techniques amplify hallucination — may apply doubly here: you're boosting reasoning through thinking mode while compressing the representations it reasons over.

**The comparison to GGUF quantization is unavoidable and revealing.** GGUF's K-quants and I-quants (see [[Choosing a GGUF Model]]) are the dominant approach to model compression for local inference, and they operate on a completely different principle — post-training quantization of existing weights vs. training with low-bit representations from the start. Bonsai's approach (train with ternary/binary constraints baked in) should theoretically preserve more quality than post-hoc quantization at equivalent bit rates, and the benchmark numbers suggest it does. But it also means you can't take an arbitrary model and Bonsai-ify it — this required training from scratch. That's the trade: better results, less fungibility.

**The phone claim matters even if you never run it on a phone.** The real significance of "fits on a phone" is that it defines a new size class. If a 27B model fits in 4 GB, it runs comfortably on any laptop, any desktop, any edge device. The iPhone constraint is a forcing function for efficiency that benefits every deployment target. This is the same dynamic that made ARM's mobile-first design win servers: optimize for the hardest target, and everything else gets the benefits for free. The harness side of that story is [[mlx-dspark]], the MLX speculative-decoding library that runs DSpark/DFlash drafters and n-gram lookup on Apple Silicon — the first to bring speculative decoding to Bonsai's hybrid linear-attention family.

**The company story is thin.** PrismML emerged from Caltech with backing from Khosla, Cerberus, and Google — plausible pedigree but no track record yet. The Bonsai family (27B, 8B, 4B, 1.7B, Image 4B) suggests a systematic approach to the size-quality spectrum, but one model release doesn't prove a research program. The real test will be whether they can sustain this across model generations as base models improve.

---

*Sources: [[raw/bonsai-27b]]*
*Last updated: 2026-07-18*
