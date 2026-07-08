# Subtext

A real-time instrument for observing the verbal workspace of an LLM as it reads, reasons, and speaks. Built on Anthropic's Jacobian lens method, it loads Qwen3.5-4B locally and streams the model's internal "silent words" — vocabulary-level readouts of residual-stream activations — to a browser canvas during live conversation. The key insight: the model's internal verdicts, plans, and intermediate concepts are visible in the lens readout several tokens before they appear in the text output.

---

## Architecture

**Server** (`server.py`, 253 lines): FastAPI + WebSocket serving a single local client. Loads Qwen3.5-4B (bf16, HuggingFace transformers) on CUDA/MPS/CPU, along with Neuronpedia's pre-fitted Jacobian lens (`neuronpedia/jacobian-lens`, revision `qwen-n1000`). For each chat turn, does a single prefill pass (KV cache enabled) with lens readouts at every new user-message token position (the *reading* phase), then token-by-token generation from the KV cache with lens readouts at each new position (the *thinking* phase). Readouts are taken at 9 layer depths spanning 12%–93% of network depth.

The core readout pipeline: `residual[layer][position] → J_l @ h → unembed → softmax → display_mask → top-k`. The J_l matrices (one per layer) are loaded from disk and kept resident on GPU for zero-cost transport. The display mask filters the 151,936-token vocabulary to ~14,000 word-start alphabetic ASCII tokens.

**Frontend** (`index.html`, 1,104 lines): Single-file HTML with inline CSS and JS. Canvas-based visualization renders each lens readout as an italic serif word positioned on horizontal rails corresponding to layer depth. Size and opacity encode probability. Amber during reading, blue during generating. Two views: a live *cloud* of all active concepts, and a *trace* view showing one word's strength across layers × tokens as a heatmap.

A scrubbable timeline stores every frame; scrubbing reconstructs exact visual state by re-simulating from frame 0. Replays (`?replay=<url>`) let anyone experience the instrument without a GPU — the hosted demo on GitHub Pages replays a pre-recorded session.

## Key techniques

**Jacobian lens transport** — The core method from Anthropic's transformer circuits research. J_l = ∂(final-layer logits) / ∂(residual at layer l). Multiplying a mid-layer residual activation by J_l projects it into the space of "what vocabulary would this activation produce if it were at the final layer?" This turns opaque internal states into interpretable word probabilities.

**Two-phase chat with KV cache reuse** — Unlike the reference implementation which does separate forward passes per position, Subtext uses a single prefill pass whose KV cache is reused for subsequent generation steps. The lens adds only a per-layer matrix-vector product and an unembedding per token, so streaming runs at native generation speed.

**Word-start display filtering** — The most polished detail. Qwen's BPE tokenizer produces fragments like "itude" (from "cert‑itude"). Subtext filters to word-start tokens only (leading space in Qwen's vocab), ASCII, alphabetic. Probabilities are computed over the full vocabulary before filtering — filtering affects legibility only, never the readout itself.

**Calligram rendering** — When a concept dominates the workspace (strength > 0.6 for 5+ frames), the system rasterizes its associated emoji and fills the silhouette with the word at random positions/sizes. Pure aesthetic, but creates a distinctive "aha" moment when a concept crystallizes mid-reasoning.

**Verification by construction** — `verify_accuracy.py` compares the live path (hooks + KV cache + transport) against the reference `JacobianLens.apply()` on identical inputs. Top-5 readouts match exactly, cosine similarity ≥ 0.99998 between logit vectors, confirming the efficiency optimizations don't distort the lens.

## Design decisions

**Real-time instrument, not a serving framework** — Makes no attempt at batching, concurrency, or multi-user support. Optimized for one person observing one model, locally. The right trade-off for a research/demo instrument.

**Single-file HTML with zero build step** — No bundler, no framework, no npm. The complexity budget goes into the visualization. This is a local-only tool; a build pipeline would add friction without benefit.

**Depth fractions, not layer indices** — The 9 visualization layers are specified as fractions of total depth, then mapped to the closest actually-fitted layers. This makes the visualization logic model-agnostic — swap the model, and the same fractions produce the same semantic spread.

**Replay as zero-friction demo** — The `?replay=` query param and hosted GitHub Pages demo let anyone see the instrument in action without a GPU. Identical visualization pipeline for live and recorded sessions.

**Non-streaming text output** — The model's actual text is only assembled after generation completes. This is deliberate: you read the lens *before* you can read the model's reply.

**Limitations**: English-only display filter (ASCII constraint); single-user only; lens reads only single-token concepts (multi-token concepts are invisible or fragmentary); layers below the fitted range are not observed.

## Comparison notes

Unlike **chain-of-thought or thinking toggles** (which show the model's self-generated reasoning tokens), Subtext shows *pre-verbal* activation patterns — concepts active in the residual stream before they're spoken, including some the model never explicitly states. This is a more direct window into model computation.

Unlike **Neuronpedia's static Jacobian lens explorer**, Subtext is conversational and continuous: it renders during live chat, includes the reading phase over user input, streams at generation speed, and provides per-token ledger and per-word inspector views.

Unlike **SAE feature dashboards** (which map static prompts to interpretable features), Subtext is a real-time instrument — closer to a debugger or oscilloscope than a static analysis tool.

Tags: #tool #project #interpretability #ai-research

---
*Sources: [[raw/subtext]]*
*Last updated: 2026-07-08*
