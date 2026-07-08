# Jacobian Lens

Anthropic's reference implementation of the Jacobian lens — a mechanistic interpretability technique that reads out what a language model's internal activations are "thinking" by linearly transporting residual-stream vectors to the final-layer basis and decoding them with the model's own vocabulary. Companion code for the paper "Verbalizable Representations Form a Global Workspace in Language Models."

---

## Architecture

The library (~600 lines of Python across 6 modules) is designed around a Protocol-based model abstraction that makes it model-library-agnostic. Any decoder transformer can be plugged in by implementing the `LensModel` Protocol (`jlens/protocol.py:19-52`): `n_layers`, `d_model`, `layers` (residual blocks), `tokenizer`, `encode()`, `forward()`, and `unembed()`. No inheritance required — structural subtyping only.

The HuggingFace adapter (`jlens/hf.py:53-62`) auto-detects the text decoder's internal layout by probing 6 known architectural families in order (Llama/Qwen/Mistral → multimodal wrappers → Phi → GPT-2 → GPT-NeoX/Pythia). A minimal `TinyDecoder` in `tests/tiny.py` satisfies the same Protocol for CPU-only testing, using `h + 0.1*linear(h)` residual blocks where the small gain keeps Jacobians well-conditioned and gives tests a mathematical ground truth to assert against (e.g., `J_2 == I + 0.1*W_3` exactly).

The fitting pipeline flows: `fit()` → `jacobian_for_prompt()` → `ActivationRecorder` hooking → `torch.autograd.grad` on retained graph → running mean accumulation → `JacobianLens`. The key computational trick: the prompt is replicated `dim_batch` times along the batch axis, so one forward pass + `ceil(d_model / dim_batch)` backward passes compute the full `[d_model, d_model]` Jacobian per layer — reusing the retained graph across backward passes rather than re-running the model.

## Key techniques

**Batch-replicated gradient estimation** (`jlens/fitting.py:100-213`). Rather than computing `∂h_final/∂h_l` one output dimension at a time, the prompt is replicated `dim_batch` times. Batch element `b` carries a one-hot cotangent at output dimension `dim_start + b`, set at every valid target position simultaneously. One backward pass computes `dim_batch` rows of each `J_l`. The retained autograd graph is reused across passes — only the cotangent changes.

**Sum-over-future estimator**. The gradient at source position `p` is `sum_{p' >= p} dh_final[p'] / dh_l[p]` — the sum over all current-and-future target positions, not just the next token. This is the reduction used in the paper; a strict per-position estimator (`dh_final[p] / dh_l[p]` averaged over `p`) gives a slightly different `J_l` but both work.

**start_graph_at trick** (`jlens/hooks.py:31-54`). When model parameters all have `requires_grad=False`, the forward pass produces no autograd graph. The recorder marks the earliest captured residual with `requires_grad_(True)`, making it the leaf that roots the graph — so the retained graph spans only from that block onward, and `torch.autograd.grad` has a graph to differentiate through.

**Chunked rank computation** (`jlens/vis.py:98-125`). Full-vocab argsort is memory-prohibitive at long sequence lengths. The `_ranks_of` function processes the sequence in chunks of 256 tokens, so peak memory is one `[chunk_size, vocab]` sort buffer rather than `[seq_len, vocab]`.

**Atomic checkpoint saves** (`jlens/fitting.py:216-221`). Checkpoints are written to a temp file then `os.replace`'d to the target path — a crash never leaves a half-written checkpoint. The `next_idx` counter is tracked separately from `n_done` so that prompts skipped as too-short are not re-processed on resume (a regression test in `test_fitting.py:206-239` verifies this).

**Two-pass visualization** (`jlens/vis.py:193-336`). `compute_slice` makes two passes over the data: first pass computes top-K per cell and discards logits; second pass re-unembeds per layer and computes ranks only for tracked tokens. This avoids holding `[seq_len, n_layers, vocab]` in memory.

## Design decisions

**Protocol over abstract class**. Using a `Protocol` rather than an ABC means any object with the right attributes works without inheritance — the HuggingFace adapter, the tiny test model, and any future model library all satisfy the interface by duck typing. This is the same pattern FastAPI popularized for dependency injection.

**Reference implementation, not a library**. The README explicitly states "not maintained and not accepting contributions." Every design choice optimizes for clarity and correctness over performance: no JIT compilation by default, no distributed training built in (users parallelize by running `fit()` on disjoint slices and merging), no streaming dataset support beyond the WikiText loader.

**fp16 save, fp32 compute**. Jacobians are computed and accumulated in fp32 but saved as fp16 — the entries are O(1) so range is not a constraint, and fp16 beats bf16 for mantissa precision on values near 1. The trade-off: half the file size (important for large models where `d_model^2` can be tens of millions of entries), negligible precision loss.

**Per-block compile, not whole-module** (`jlens/hf.py:141-144`). `torch.compile` wraps individual residual blocks, not the entire model. Whole-module compilation would inline blocks and bypass forward hooks — per-block preserves hook boundaries so `ActivationRecorder` still fires and the retained graph is bounded per block.

**Default BOS forcing** (`jlens/hf.py:105-110`). Many instruction-tuned checkpoints ship with `add_bos_token=False`. The adapter forces it to `True` by default because raw-text prompts degrade without an attention-sink BOS token. Callers can opt out with `force_bos=False`.

**First 16 positions excluded** (`jlens/fitting.py:42`). Early positions in the sequence act as attention sinks with atypical residual statistics — including them would contaminate the Jacobian estimate with positional artifacts. The final position is also excluded (no next-token target). This means each 128-token prompt contributes at most 111 valid positions.

**Model's own unembedding as the decoder**. Rather than training a separate probe or classifier, the lens uses the model's own `unembed()` (final norm + LM head) to decode transported residuals. This is what makes the output interpretable as "what the model would say" rather than an abstract feature — the tokens are actual vocabulary items the model knows.

## Comparison notes

Unlike **sparse autoencoders (SAEs)** which decompose activations into a learned dictionary of monosemantic features, the Jacobian lens requires no additional training beyond computing `J_l`. It uses the model's existing vocabulary as its feature basis, trading SAEs' ability to find novel features for immediate interpretability and zero additional parameters.

Unlike the **logit lens** (which simply unembeds each layer's residual without transport), the Jacobian lens accounts for the fact that the residual stream rotates through representation space across layers. The `J_l` matrix is effectively a change-of-basis that makes an early-layer residual "speak the language" of the final layer's unembedding. The walkthrough notebook demonstrates this: at mid layers where the logit lens is still noise, the J-lens surfaces interpretable tokens.

The project is in a lineage of Anthropic's transformer-circuits research ([[Emotion concepts and their function in a large language model]] for the feature-analysis side, [[A Non-Anthropomorphized View of LLMs]] for the philosophy). The key conceptual contribution is the **global workspace** thesis: language models develop a shared representational space at mid-network layers where verbalizable content converges — analogous to the Global Workspace Theory of human consciousness.

Unlike many interpretability tools that focus on *identifying* features, the Jacobian lens focuses on *reading out* what the model is disposed to say at any point in its computation. This makes it a tool for studying not just what a model knows, but *when* it knows it — the temporal dynamics of computation through the layer stack.

## Tags

#tool #project #interpretability #transformers #mechanistic-interpretability

---
*Sources: [[raw/jacobian-lens]]*
*Last updated: 2026-07-08*
