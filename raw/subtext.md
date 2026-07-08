---
url: https://github.com/ninjahawk/Subtext
title: Subtext — Live Jacobian-Lens Thought Streaming
author: ninjahawk
date_fetched: 2026-07-08
date_published: 2026-06
---

# Subtext

A real-time instrument for observing the verbal workspace of a language model as it reads, reasons, and speaks. Built by ninjahawk, independent of Anthropic but using their Jacobian lens method. ~1,555 lines total: 253 line Python server, 1,104 line single-file HTML/JS frontend, plus tests and verification.

## What it does

Loads Qwen3.5-4B locally (bf16, HuggingFace transformers) with a pre-fitted Jacobian lens from Neuronpedia. Serves a single-page web app via FastAPI + WebSocket on localhost:8765. For every conversation turn:

1. **Reading phase**: Prefill pass over the user's message, lens read at every token position
2. **Thinking phase**: Token-by-token generation with KV cache, lens read at the newest position each step

At each of 9 layer depths (fractions 0.12 through 0.93 of total depth), the residual stream activation is transported through the Jacobian lens matrix J_l into the final-layer basis, unembedded to vocabulary logits, softmaxed, and the top-k word tokens are streamed to the browser as a "thoughts" frame.

## Architecture

```
browser (single HTML file)  ⇐ websocket ⇐  server.py
    Qwen3.5-4B (bf16, HF transformers, KV cache)
    pre-fitted Jacobian lens: neuronpedia/jacobian-lens, revision qwen-n1000
    per token: residual hooks at 9 layers → J_l transport → unembed
             → full-vocabulary softmax → word-start top-k → frame
```

### Server (`server.py`, 253 lines)

- **Model loading** (lines 39-53): Auto-detects CUDA → MPS → CPU, uses bf16 on GPU, fp32 on CPU. Loads Qwen3.5-4B via HuggingFace transformers, wraps with jlens.from_hf().
- **Lens loading** (lines 55-68): Loads pre-fitted JacobianLens from neuronpedia/jacobian-lens (revision `qwen-n1000`). Selects the 9 closest fitted layers to the requested depth fractions, keeps only those Jacobians resident on GPU.
- **Display mask** (lines 70-90): Builds/caches a boolean mask over the vocabulary. Starts with jlens's own `_meaningful_token_mask()`, then further restricts to word-START tokens only (Qwen's tokenizer prefixes continuations with a space — e.g. "itude" from "cert-itude" gets filtered), ASCII, alphanumeric with hyphens/apostrophes, length > 2. The paper's own filter, extended for legibility.
- **ResidualCatcher** (lines 93-111): Registers forward hooks on the chosen decoder blocks. Stores detached hidden states per layer.
- **readout()** (lines 114-147): The core lens function. For each viz layer: extract residual at position, transport via `lens.transport(h, layer)`, unembed via `model.unembed()`, softmax, mask non-display tokens, take top-k (8 per layer). Then aggregate across layers: per-word max probability, probability-weighted mean depth, per-layer profile. Returns top 14 words per frame.
- **Chat endpoint** (lines 175-247): WebSocket at /ws. Receives chat messages, tokenizes with chat template (enable_thinking=False for Qwen's thinking toggles), does prefill with use_cache=True, extracts KV cache, sends reading frames for user-message token positions, then generates token-by-token with top-p 0.95 sampling, sending thinking frames. Closes with a "done" message containing the full decoded reply.
- **Two-phase boundary** (lines 158-164): `encode_chat()` tokenizes the full conversation with the assistant prefix, then locates where the newest user message starts by comparing against the previous conversation without that message — so the reading phase only covers newly typed tokens, not the full history.

### Frontend (`index.html`, 1,104 lines)

Single-file HTML with inline CSS and JS. No build step, no dependencies beyond Google Fonts.

**Visualization canvas** (`field` object, lines 443-609):
- **Cloud view**: Words rendered as italic serif text on horizontal rails corresponding to layer depth. Size and opacity encode probability (power transform 0.45, scale 1.05). Amber during reading, blue during generating. Words fade with 0.78 decay factor per frame, target tracks max seen probability. Only the top 10 visible words survive the pruning pass.
- **Calligrams**: When a word with a known emoji (from a ~400-entry map) dominates at strength > 0.6 for 5+ consecutive frames, the system builds a "calligram" — the emoji shape filled with the word itself at various sizes — rendered as an overlay. Throttled to once per 25 seconds per word.
- **Relaxation**: 4-iteration force-directed horizontal separation of overlapping words on nearby rails.
- **Trace view**: Layer × token heatmap for a single word. X-axis = token position (shared with scrubber), Y-axis = layer, brightness = lens strength. Reveals concepts climbing the layer stack before being spoken.
- **Hover/click**: Hover shows per-layer activation profile as a vertical dotted line with circles; click opens an inspector with peak strength, mean depth, strength history sparkline.

**Right panel** (console, lines 728-855):
- Chat log with character-by-character read highlighting (user message tokens highlighted in amber as they're read)
- Live "top of mind" ranking (top 7 words by current strength)
- Per-token ledger: every frame as a row showing the input/output token plus top 3 thoughts, clickable to scrub to that moment
- Word statistics table: aggregated across the entire response — every distinct word ranked by "presence" (how many tokens it was active), with peak layer and count. Click to jump to peak moment and switch to trace view.

**Timeline player** (lines 857-1000):
- All frames stored in `frames[]` array. Scrubbing reconstructs state by re-simulating `field.applyFrame()` from frame 0 — fast because it's just a few hundred frames of simple arithmetic.
- Transport controls: step, play/pause, scrubber, speed (0.5×/1×/2×/4×), live button to catch up to generation edge.
- During generation, the view rides the live edge; scrubbing back pauses and lets you inspect past moments.

**Replay** (lines 1061-1092): `?replay=<url>` query param loads a JSON export and replays with live pacing. The hosted demo at ninjahawk.github.io/Subtext replays a pre-recorded demo session. No GPU needed for replay.

**Session export**: ⤓ button downloads all frames as JSON for sharing/replay.

### Tests and verification

- **verify_accuracy.py** (81 lines): Compares Subtext's live path (forward hooks + KV cache + transport + unembed) against the reference `JacobianLens.apply()` on identical inputs. Verifies that top-5 readouts match and cosine similarity ≥ 0.99998 across 4 layers × 3 positions on the walkthrough prompt. Includes the expected two-hop intermediates (Italy → euros).
- **test_client.py** (36 lines): WebSocket smoke test — sends "Is this correct? 12 + 5 = 1", prints samples of reading and thinking frames.
- **test_multiturn.py** (43 lines): Two-turn conversation test verifying history handling: sends name, asks for recall, checks that "Nate" appears in the reply.
- **record_session.py** (38 lines): Records a demo session to docs/demo_session.json for GitHub Pages replay.

## Key techniques

### Jacobian lens transport

The Jacobian lens method (from Anthropic's transformer circuits paper) works by computing J_l = ∂(unembed(output at final layer)) / ∂(residual at layer l) — that is, how the final-layer vocabulary logits would change given a change at layer l. Multiplying a residual-stream activation at layer l by J_l transports it into the "final-layer basis," answering: "if this internal state were at the final layer, what vocabulary words would it produce?"

The implementation stores pre-computed J_l matrices per layer (loaded from Neuronpedia's pre-fitted lens), and the `readout()` function performs:
```
h_l = residual[l][position]
transported = J_l @ h_l          # matrix-vector product
logits = unembed(transported)    # vocabulary projection
probs = softmax(logits)
probs[~display_mask] = 0         # filter fragments/punctuation
top_k_words = topk(probs, 8)     # per layer
```

### Two-phase chat with KV cache reuse

Unlike the reference implementation which does separate forward passes per position, Subtext uses the model's KV cache to share computation:

1. One prefill pass with `use_cache=True` processes the entire prompt
2. The KV cache from that pass (`out.past_key_values`) is preserved
3. Reading-phase lens readouts are taken from the residual hooks at each position of the user's newest message
4. Generation-phase uses the cached KV state for all previous tokens, only computing one new token per step

This means streaming runs at native generation speed — the lens adds only a per-layer matrix-vector product and an unembedding per token.

### Word-start display filtering

The display vocabulary is aggressive: it starts from jlens's own `_meaningful_token_mask()` (which removes punctuation, numbers, etc.), then further filters to tokens that:
- Start with a space (word-start in Qwen's tokenizer — filters BPE continuations like "itude")
- Are pure ASCII (so the stream reads in English)
- Match `^[A-Za-z][A-Za-z'\-]+$` (alphabetic with hyphens/apostrophes)
- Have length > 2

This matters because the raw top-k from the lens contains BPE fragments ("itude" from "cert‑itude") and punctuation. The reference implementation's `mask_display` does something similar, but Subtext's word-start criterion is stricter. Critically, probabilities are computed over the full vocabulary before any filtering — filtering affects legibility only, never the readout itself.

### Calligram generation

An unusual visualization technique: when a word dominates the workspace (strength > 0.6 for 5+ consecutive frames) and has a known emoji, the system rasterizes the emoji at 340px, extracts its alpha channel at 3px grid resolution, then uses rejection sampling to place the word at random positions within the emoji's silhouette at progressively smaller font sizes (36px down to 9px). The result renders on the canvas as the word repeated in the shape of its associated emoji. This is purely aesthetic but creates a distinctive "aha" moment when a concept crystallizes.

### Scrubbable timeline with snap reconstruction

Live playback is fast; every frame is stored in memory. Scrubbing to any token position reconstructs the exact visual state by re-running `field.applyFrame()` from frame 0 (fast — just probability updates and relaxation on a few hundred frames). Words are "snapped" to their target strength (no animation interpolation) so the frozen frame is exact.

## Design decisions

### Optimized for real-time visualization, not model serving

Subtext is not an LLM serving framework. It makes no attempt at batching, concurrency, or production serving. Everything is optimized for a single interactive session: one model, one user, one WebSocket connection at a time. This simplicity is the right call — the value is in the visualization, not the serving infrastructure.

### bf16 on GPU, fp32 on CPU with auto-detection

The server auto-detects CUDA → MPS → CPU and selects dtype accordingly. bf16 matmuls are slow/patchy on CPU, so it falls back to fp32 there. This means the code works on NVIDIA GPUs (~10GB VRAM), Apple Silicon (16GB+ unified memory), and CPU (slow but usable for smoke tests) without configuration changes.

### Layer depth fractions, not layer indices

The 9 visualization layers are specified as fractions of network depth (0.12, 0.25, 0.35...0.93), then mapped to the closest actually-fitted layers in the pre-computed lens. This means the visualization logic is independent of the specific model architecture — swap the model, and the same fractions produce the same semantic spread (early perception through mid-workspace to near-emission).

### Single-file HTML — zero build step

The entire frontend is one HTML file. No JavaScript bundler, no framework, no npm install. Inline CSS, inline JS, Google Fonts CDN. This is a deliberate choice: the app is a local-only instrument, not a deployed service. The complexity budget goes into the visualization, not into build tooling.

### Replay as distribution mechanism

By supporting `?replay=<url>` and hosting a demo session on GitHub Pages, the project lets anyone see the instrument in action without a GPU. The replay path is the same as the live path — frames flow through the same `ingestFrame()` / `advanceOne()` pipeline — so the visualization is identical whether live or recorded. This is a clever zero-friction demo strategy.

### Non-streaming display of model output

While the lens streams at the model's generation speed, the model's actual text output is only shown after generation completes (the "done" message contains the full decoded text). During generation, output tokens are accumulated character-by-character into the chat bubble. This means you can read the lens before you can read the model's reply — which is the entire point of the instrument.

### Weakness: no concurrency, no sessions

The server handles one WebSocket at a time with a simple accept-receive loop. No session management, no concurrent users. This is fine for the intended use case (personal instrument on localhost) but would need significant rework for any multi-user scenario.

### Weakness: display filter is English-only

The ASCII + alphabetic filter means non-English readouts are invisible. The justification is that the Qwen model's lens was fitted on English data and non-English tokens in the workspace would be rare/fragmentary, but this is a limitation worth noting.

## Comparison to related work

### vs. Neuronpedia's interactive Jacobian lens demo

Neuronpedia hosts a web-based Jacobian lens explorer that lets you type prompts and see readouts. Subtext differs in being conversational and continuous: it renders the lens during a live chat, includes the reading phase over the user's message, streams at generation speed via KV cache, and pairs the canvas with a per-token ledger and per-word inspector. The Neuronpedia demo is a snapshot explorer; Subtext is a real-time instrument.

### vs. Anthropic's reference implementation (jlens)

The reference implementation (`jlens.JacobianLens.apply()`) does separate forward passes per position/layer. Subtext's live path uses forward hooks + KV cache for efficiency, and `verify_accuracy.py` confirms the outputs are equivalent (cosine similarity ≥ 0.99998). The reference implementation is a research tool; Subtext wraps it in a real-time visualization.

### vs. mechanistic interpretability tools (SAE dashboards, Neuronpedia)

Most interpretability tools focus on static analysis: upload a prompt, see feature activations, explore. Subtext is dynamic: it shows the model's internal state evolving in real time during conversation. It's closer to a debugger or oscilloscope than a static analysis tool. The innovation is not the lens method itself but the continuous, conversational application of it.

### vs. chain-of-thought / thinking toggles

Some models (including Qwen 3.5) expose a "thinking" mode where the model outputs reasoning tokens before the answer. Subtext shows something different: not the model's self-generated reasoning, but the *pre-verbal* activation patterns in the residual stream — concepts that are active before they're spoken, including some the model never explicitly states. This is a different, more direct window into model computation.

## Dependencies

```
torch>=2.5
transformers>=4.56
fastapi
uvicorn[standard]
websockets
huggingface_hub
jlens @ git+https://github.com/anthropics/jacobian-lens
```

Total: ~9 GB download (model + lens), ~10 GB VRAM for GPU inference, 16 GB unified memory for Apple Silicon.

## References

- Anthropic's Jacobian lens paper: https://transformer-circuits.pub/2026/workspace/index.html
- Reference implementation: https://github.com/anthropics/jacobian-lens
- Pre-fitted lens weights: https://huggingface.co/neuronpedia/jacobian-lens
- Qwen3.5-4B: https://huggingface.co/Qwen/Qwen3.5-4B
