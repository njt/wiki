---
url: https://github.com/anthropics/jacobian-lens
title: jlens — Jacobian lens
author: Anthropic PBC
date_fetched: 2026-07-08
date_published: 2026-06
---

# jlens — Jacobian lens

Reference implementation for the paper **"Verbalizable Representations Form a Global Workspace in Language Models"** (https://transformer-circuits.pub/2026/workspace/index.html).

> **Reference implementation.** Not maintained and not accepting contributions.

## What it does

The Jacobian lens reads out what an internal activation is disposed to make the model say. It linearly transports a residual-stream vector at any layer and position into the final-layer basis, then decodes it with the model's own unembedding into a ranked list of vocabulary tokens.

Core formula:
```
lens_l(h) = unembed( J_l @ h ), J_l = E[∂h_final / ∂h_l]
```

The expectation is over prompts, source positions, and all current-and-future target positions in a generic web-text corpus.

## Project structure

```
jlens/
  __init__.py      — public API surface (12 exports)
  lens.py          — JacobianLens class: apply, transport, save/load, merge
  fitting.py       — fit(), jacobian_for_prompt(): the Jacobian estimator
  protocol.py      — LensModel Protocol: model-agnostic interface
  hooks.py         — ActivationRecorder: forward-hook context manager
  hf.py            — HuggingFace adapter (HFLensModel, from_hf, Layout)
  vis.py           — Interactive slice visualization (compute_slice, build_page)
  examples.py      — Bundled example prompts, WikiText loader
  _logging.py      — stderr logging with elapsed/delta timestamps
  data/
    slice_vis.html — D3-based interactive HTML template
    blackmail.json — Agentic Misalignment scenario prompt
tests/
  tiny.py                    — Tiny CPU decoder (4-layer, d_model=8) for tests
  test_fitting.py            — Jacobian estimation, fit/save/load round-trip, merge, resume
  test_compute_slice.py      — Visualization computation end-to-end
  test_ranks_of.py           — Chunked rank computation correctness
  test_vis_modes.py          — Page rendering (embed vs fetch modes)
  test_hf_layout.py          — HF model layout auto-detection
data/
  evaluations/  — 6 prompt distributions for lens-quality evaluation
  experiments/  — 11 experiment prompt sets (probe-swap, verbal-introspection, etc.)
```

## Architecture

The project is ~600 lines of Python across 6 core modules, designed as a model-agnostic library through a Protocol-based interface.

### Protocol-based model abstraction

The central design pattern: `LensModel` is a `Protocol` (structural subtyping, no inheritance required). Any model can be plugged in by implementing:
- `n_layers`, `d_model` — model dimensions
- `layers` — sequence of residual blocks (forward-hook targets)
- `tokenizer` — tokenizer with `.decode()` method
- `encode(text)` — tokenize to input_ids tensor
- `forward(input_ids)` — run residual stack (no LM head), must build autograd graph
- `unembed(residual)` — final norm + LM head, maps [..., d_model] → [..., vocab_size]

The HuggingFace adapter (`hf.py`) implements this by auto-detecting the layout of the text decoder inside common HF architecture families (Llama, Qwen, Mistral, Gemma, OLMo, Phi, GPT-2, GPT-NeoX/Pythia). `tests/tiny.py` provides a minimal from-scratch `TinyDecoder` that satisfies `LensModel` for testing without transformers.

### Fitting algorithm

The lens fitting (`fitting.py`) computes the average input-output Jacobian:

1. **Per-prompt estimator** (`jacobian_for_prompt`):
   - Replicate the prompt `dim_batch` times along the batch axis
   - Run one forward pass, retain the autograd graph
   - Loop `ceil(d_model / dim_batch)` times:
     - Inject a one-hot cotangent at every valid target position for `dim_batch` output dimensions
     - Call `torch.autograd.grad` with `retain_graph=True` (except last pass)
     - Each backward computes `dim_batch` rows of `J_l` at once
   - The gradient at source position p is `sum_{p' >= p} dh_final[p'] / dh_l[p]` — the sum over all later target positions

2. **Multi-prompt accumulation** (`fit`):
   - Iterates over prompts, accumulates per-prompt Jacobians as a running mean
   - Supports checkpointing (atomic saves, resume on restart)
   - Tracks per-prompt diagnostics: Jacobian norm (flags heavy-tailed outliers) and relative shift in running mean (tracks convergence)
   - Skips too-short prompts elegantly (tracks `next_idx` separately from `n_done` to avoid double-counting on resume)
   - `merge()` classmethod combines lenses fitted on disjoint prompt subsets via `n_prompts`-weighted mean

### Activation recording

`ActivationRecorder` (`hooks.py`) is a context manager that registers forward hooks on specific residual blocks. Key design:
- Hooks store output tensors without detaching them, so they can be used directly in `torch.autograd.grad`
- `start_graph_at` option marks one captured tensor with `requires_grad_(True)`, making it the leaf that roots the autograd graph — essential when model parameters have `requires_grad=False`
- Handles HF blocks that return tuples (hidden, present_kv, ...) by extracting the first tensor
- Cleans up hooks on `__exit__`, with rollback on failed `__enter__`

### Interactive visualization

`vis.py` produces a position × layer HTML heatmap showing the lens's top-1 token at each cell, with:
- Superscript showing rank over the full vocabulary
- Click-to-pin tokens with rank-tracking charts
- Bottom row showing the model's actual output (J=I at final layer)
- Two modes: `"embed"` (self-contained single file, d3 inlined, rank data base64-embedded) and `"fetch"` (sidecar files, lazy loading, CDN d3)
- Chunked rank computation to avoid full-vocab argsort at long sequence lengths
- `mask_display` option to restrict displayed tokens to word-like ones (ranks remain full-vocab)
- English gloss support for non-Latin tokens (`alt_token`)

### Computation patterns

- **Batch replication trick**: The prompt is replicated `dim_batch` times so one backward pass computes `dim_batch` rows of each `J_l` simultaneously
- **Graph retention**: One forward pass, many backward passes on the retained graph — avoids re-running the model for each chunk of output dimensions
- **Chunked sorting**: Rank computation processes the sequence in chunks of `chunk_size` so peak memory is one `[chunk_size, vocab]` sort buffer
- **Memory-efficient visualization**: Two-pass approach — first pass computes top-K (discarding logits), second pass re-unembeds per layer and computes ranks only for tracked tokens

## Implementation details

- The estimator sums over all later target positions (`p' >= p`), not just the next token — this is the reduction used in the paper
- Early positions (first 16) are excluded as attention sinks with atypical residual statistics
- The final position is excluded because it has no next-token target
- Jacobians are stored as `dict[int, Tensor[d_model, d_model]]` — one `[d_model, d_model]` matrix per layer
- Save format uses fp16 (halves file size; entries are O(1) so range is not a constraint, and fp16's extra mantissa bits beat bf16)
- Fitting quality saturates quickly: ~100 prompts is usable, 1000 produces paper-quality lenses
- Fitting cost: one forward pass + `ceil(d_model / dim_batch)` backward passes per prompt
- Can parallelize by running `fit()` on disjoint prompt slices and merging with `JacobianLens.merge()`
- Supports `torch.compile` per-block (not whole-module, which would bypass hooks)
- Handles logit softcapping (Gemma) in the unembed path

## Test infrastructure

- `tests/tiny.py`: A 4-layer CPU decoder with `h + 0.1 * linear(h)` residual blocks. The small gain keeps the Jacobian well-conditioned, and the block structure means `J_2 == I + 0.1*W_3` exactly — giving tests a mathematical ground truth to assert against
- `tests/test_fitting.py`: 10 tests covering the full fitting pipeline including a regression test for a desync bug where skipped prompts caused double-counting on resume
- `tests/test_compute_slice.py`: End-to-end visualization tests including tracked rank columns matching `_ranks_of` output

## Dependencies

- `torch` — core computation
- `transformers >= 5.5` — HuggingFace model loading
- `huggingface_hub` — downloading pre-fitted lenses from the Hub
- `numpy` — visualization array operations
- Optional: `datasets` (WikiText loading), `pytest`, `ruff`

## Data included

- 6 evaluation prompt sets (multihop, multilingual, poetry, order-ops, association, typo) — synthetic, authored by Anthropic, Apache 2.0
- 11 experiment prompt sets covering the global-workspace paper's experiments (probe-swap, verbal-introspection, verbal-report, directed-modulation, top-down-summoning, flexible-generalization, selectivity, ignition, capacity, dual-task)
- 8 bundled example prompts in `jlens.examples.EXAMPLES`
- Slice visualization uses d3 (ISC license) from jsDelivr CDN or inlined

No model weights or text corpora are bundled.
