---
url: https://github.com/anthropics/jacobian-lens
title: "jlens — Jacobian lens"
author: Anthropic PBC
date_fetched: 2026-07-08
date_published: 2026-06
---

Reference implementation for the paper "Verbalizable Representations Form a
Global Workspace in Language Models." The Jacobian lens reads out what an
internal activation is disposed to make the model say: it linearly transports
a residual-stream vector at any layer and position into the final-layer basis,
then decodes it with the model's unembedding into a ranked list of vocabulary
tokens. Core formula: `lens_l(h) = unembed( J_l @ h )` where `J_l` is the
expected input-output Jacobian `E[∂h_final / ∂h_l]` over a generic web-text
corpus.

The ~600-line Python library is model-agnostic through a Protocol-based
interface (`LensModel`). Any model can be plugged in by implementing a handful
of methods; a HuggingFace adapter handles auto-detection across common
architectures (Llama, Qwen, Mistral, Gemma, OLMo, Phi, GPT-2, Pythia). Fitting
computes the average Jacobian via one forward pass plus
`ceil(d_model / dim_batch)` backward passes per prompt, summing over all later
target positions at each source position. The project includes an interactive
D3-based heatmap visualization showing the lens's top-1 token per position and
layer, with click-to-pin rank tracking.

It is not maintained and not accepting contributions. Bundled data includes 6
evaluation prompt sets, 11 experiment prompt sets from the paper, and 8 example
prompts — all synthetic, Apache 2.0 licensed. No model weights or text corpora
are included.
