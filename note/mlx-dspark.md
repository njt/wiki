# mlx-dspark

mlx-dspark is the most complete speculative-decoding library for Apple Silicon — an MIT-licensed MLX toolkit that runs EAGLE-style (DSpark) and block-diffusion (DFlash) drafters, plus a drafter-free n-gram lookup mode, all behind a verify loop that is lossless by construction and a hardware-aware auto-calibration layer that retunes the draft length for each specific Mac. It is the piece that lets models like [[Bonsai 27B]] and [[Muse Glimmer]]'s DFlash drafter actually deliver their speedups on M-series hardware.

---

## Architecture

The design separates **proposal** from **verification** and makes only the latter authoritative.

- **The verify loop** (`generate.py`) is the invariant everything hangs off. Each round drafts m tokens, the target forward runs `[anchor] + m` draft tokens in one pass, the draft is compared against the target's own logits, and the longest accepted prefix is committed. The target — not the drafter — decides every token, which is what makes greedy output *equal* plain decoding (up to fp ties) and makes temperature > 0 an exact sample via the Leviathan/Chen accept rule `min(1, p/q)`.
- **Three proposal sources** feed the same loop: the **DSpark** drafter (`model.py`, a 5-layer backbone + rank-256 Markov head + confidence head, cross-attending from the current block over `[context, block]`), the **DFlash** block-diffusion drafter (denoises a 16-token block in parallel, reusing the target's embedding and `lm_head`), and **lookup** (`lookup.py`, a pure n-gram index — no model, no hidden-state tap).
- **The target wrapper** (`target.py`) hides the family differences behind one interface: `make_cache()`, `run(ids, cache, tap)`, `verify()`/`rollback()`. Dense families trim the KV cache on rejection; hybrid families can't, because linear-attention state advances through every token a forward touches.
- **Calibration** (`calibrate.py`) sits outside the loop: it measures the verify and drafter cost curves once per (chip × MLX version × quantization × model) pair, caches them to disk, and hands a `CapController` the numbers to pick each round's draft cap.

## Key techniques

- **Losslessness by construction.** The drafter proposes, the target disposes. No quality knob, no approximation — the output *is* the target's output, just reached in fewer sequential steps. This is the same guarantee the whole EAGLE lineage makes, enforced by the target-verify structure rather than by training luck.
- **The Apple-Silicon verify-cost curve.** Multi-token verification leaves quantized matmul's cheap few-rows path; the cost of a wider round has a "knee" that is a property of (chip × MLX version × quantization), not a constant. The cost model `tok/s ≈ A / (drafter + overhead + slope·C)` captures it, and the library *measures* rather than assumes — a static cap tuned on an M4 Max is wrong on an M1 Air.
- **Hardware-aware auto-calibration.** `CapController` picks the draft cap per round from the cached curves plus a live acceptance EWMA (`expected_committed = 1 + Σ p^i`), and drops to a cap-0 "parked sprint" when speculation is losing. It's the same idea as adaptive speculation elsewhere, but grounded in measured on-device costs.
- **Custom kernels for the two ends of the curve.** A vendored small-M `simdgroup_matrix` MMA verify kernel dequantizes each weight group once and reuses it across rows (flattening widths 6–8 to ~width-5 cost); a wide-GEMM prefill dequantizes once for the whole chunk; `PREFILL_LAST_ROW_HEAD` projects only the final prefill row to vocab (~7–8% of prefill FLOPs saved).
- **Capture-and-rerun rollback for hybrid targets.** Recurrent state (gated-DeltaNet in Bonsai/Qwen3.6, Mamba-2 in Nemotron-H) has no trim. mlx-dspark records each verify round's recurrence inputs via scoped hooks, then rebuilds the state at the accept point bit-for-bit — rejects cost ~48 tiny recurrence kernels instead of re-forwarding the accepted prefix through the whole model.
- **A faithfulness probe at load.** `verify_tap()` reproduces the model's own logits on a tiny input and fails loudly if the replicated hidden-state tap diverges, refusing windowed attention it can't reproduce rather than drafting from a silently-wrong stream.

## Design decisions

- **Speculative decoding is a latency tool, not a throughput tool** — and mlx-dspark leans in. On a memory-bandwidth-bound single stream (see [[Theoretical LLM Inference Bottlenecks]]), speculation trades spare compute for fewer sequential steps. The library's continuous batching and MoE notes concede the inverse: at batch, the spare compute doesn't exist and speculation and batching are substitutes, not complements.
- **Calibrate once, cache on disk.** Measuring verify/drafter costs per pair costs ~5s and the result is keyed by device + MLX version + mode + model basenames, so a version bump or a different quant invalidates cleanly. This is the honest alternative to hardcoded caps — and it only works because the target model's output is unchanged by the cap, so recalibration carries zero quality risk.
- **Drafter-free is a first-class mode.** Lookup drafting needs no tap, no extra weights, and a miss costs a plain 1-token forward (zero miss cost), which makes it the lowest-risk speedup for RAG quoting, code editing, and summarization — exactly the copy-heavy workloads where the continuation already exists earlier in context.
- **The proposal head must not inherit the verifier's output transform.** On Muse-Glimmer, applying the target's `output_multiplier` + logit-softcapping to a draft proposal shrinks the base logits so the Markov bias overwhelms them and d0 collapses ~85% → ~29%. The reuse path returns *raw* `lm_head` logits while verification keeps the full head — a subtle but load-bearing distinction.
- **Fail loudly, not silently.** The tap probe, the windowed-attention refusal, and the KV-quantization guards all choose an explicit error over a plausible-looking wrong output.

## Comparison notes

- **Against vLLM's EAGLE / Medusa on CUDA:** the drafter families are the same lineage (EAGLE-style semi-autoregressive, EAGLE-3's reduced draft vocab), but mlx-dspark's distinguishing work is the Apple-Silicon cost model — the verify-width knee is a *device* property, and no CUDA serving stack needs on-device calibration the way MLX's quantized matmul does.
- **Against [[Inference Cost Napkin Math]] and [[Theoretical LLM Inference Bottlenecks]]:** those pages derive *why* decode is bandwidth-bound and why speculation is a latency (not throughput) tool; mlx-dspark is the concrete Apple-Silicon instantiation — it measures the exact verify-cost curve those derivations predict and turns it into a per-round tuning policy. It also complicates the "speculation reduces aggregate throughput at batch" claim by *choosing* to specialize on the single-stream, batch-1 regime where the spare compute genuinely exists.
- **Against [[Bonsai 27B]]:** Bonsai is the model that makes a 27B phone-viable through extreme quantization; mlx-dspark is the harness that makes *generation* fast on the Apple hardware Bonsai targets, including the first speculative decoding for Bonsai's hybrid linear-attention family on Apple Silicon. The two are complementary halves of the same local-inference story.
- **Against [[Muse Glimmer]]:** Muse-Glimmer ships a DFlash drafter as a product feature; mlx-dspark is where that drafter actually runs on a Mac — and its `muse_glimmer` target path exists precisely because Meta's VLM ships no hidden-state capture hook, forcing a replicated forward + faithfulness probe that denser families get for free.

---

*Sources: [[raw/mlx-dspark]], [[summary/mlx-dspark]]*
*Last updated: 2026-08-21*
