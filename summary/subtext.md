---
url: https://github.com/ninjahawk/Subtext
title: "Subtext — Live Jacobian-Lens Thought Streaming"
author: ninjahawk
date_fetched: 2026-07-08
date_published: 2026-06
topics:
  - developer-tools
---

Subtext is a real-time instrument for observing the pre-verbal workspace of a
language model as it reads a message and generates a reply. It loads
Qwen3.5-4B locally with a pre-fitted Jacobian lens from Neuronpedia and serves
a single-page web app over WebSocket on localhost:8765.

For each conversation turn it runs two phases. During reading, it does a
prefill pass over the user's new tokens, reading the lens at every position.
During thinking, it generates token-by-token with KV-cache reuse, reading the
lens at each new position. At nine layer depths spanning 0.12 through 0.93 of
network depth, each residual-stream activation is transported through the
Jacobian matrix into the final-layer vocabulary basis, unembedded to logits,
softmaxed, and filtered to word-start tokens only — producing top-k "thoughts"
per layer. The display filter strips BPE fragments, punctuation, and non-ASCII
tokens so the stream is legible English words.

The frontend (a single 1,104-line HTML file with no build step or framework)
renders these thoughts on a canvas as words sized and positioned by probability
across layer-depth rails — amber during reading, blue during generation. A
calligram system renders words in the shape of their associated emoji when a
concept dominates for several frames. A timeline player stores every frame and
supports scrubbing with snap reconstruction; a per-token ledger records every
thought frame alongside the conversation. Session export and replay are built
in, and a hosted demo replays a pre-recorded session on GitHub Pages with no
GPU needed.

The architecture is optimized for a single-user instrument, not model serving
— no batching, no concurrency, no session management. It auto-detects CUDA,
MPS, or CPU and selects dtype accordingly, running on NVIDIA GPUs (~10 GB
VRAM), Apple Silicon (16 GB+ unified memory), or CPU. Accuracy verification
confirms cosine similarity ≥ 0.99998 against the reference Jacobian lens
implementation.

Subtext differs from static interpretability tools by being continuous and
conversational. It is closer to a debugger or oscilloscope than a snapshot
explorer: the value is in watching concepts activate, strengthen, and fade
across the layer stack in real time — including concepts the model never
explicitly states.
