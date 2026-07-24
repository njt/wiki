# Petals — Decentralized LLM Inference

Petals is a BigScience research project that lets anyone run 100B+ parameter language models on consumer hardware by splitting the model across a peer-to-peer network of volunteer GPUs. Think SETI@home or BitTorrent, but for LLM inference: you load a slice of the model, other peers serve the rest, and together you get ~6 tok/s on Llama 2 70B — enough for conversational use. It's the most radical answer to the "how do I run big models without a datacenter?" question.

---

## How It Works

Each participant downloads a portion of a model's layers (a "server block"). When someone runs inference, their client routes each layer through the network to whichever peer is serving that block. You only need enough VRAM for your assigned slice — a single consumer GPU can serve a fraction of a 176B model.

The network status is publicly visible at `health.petals.dev`, giving it a transparency most distributed systems lack. Supported models include Llama 3.1 (405B), Mixtral 8x22B, Falcon 40B+, and BLOOM 176B. Both inference and fine-tuning are supported, and it works from Google Colab.

## Key Quotes

> "Run large language models at home, BitTorrent‑style."

The pitch is exactly right. This isn't another API wrapper — it's a fundamentally different resource model. Instead of paying per token, you contribute compute and get inference in return.

> "Combine the comforts of an API with the flexibility of PyTorch and 🤗 Transformers."

This is the design tension Petals sits inside: it wants to feel like calling `model.generate()` while actually dispatching across a P2P network. The API surface is deceptively simple; the coordination underneath is the hard part.

> Users can "employ any fine-tuning and sampling methods, execute custom paths through the model, or see its hidden states."

This is what a P2P architecture uniquely enables that a hosted API structurally cannot: arbitrary model introspection. You can't inspect hidden states on OpenAI's servers. You can when the computation happens on your machine and your peers'.

## Key Themes

#distributed-systems #p2p #local-inference #open-source-ai #model-serving #bigscience #volunteer-computing

## Critical Analysis

**The volunteer reliability problem is the elephant in the room.** The page benchmarks ~6 tok/s for a 70B model. That's barely conversational speed, and it assumes all peers are online, responsive, and serving at full throughput. In practice, a P2P network of consumer GPUs will have churn, heterogeneous hardware, and variable network latency. The gap between "works in the lab" and "works when you need it" is where most P2P systems die.

**It's a research artifact, not a product.** Petals emerged from BigScience, the same workshop that produced BLOOM. It proved the concept. But the page hasn't meaningfully evolved — the TechCrunch coverage is from December 2022, and the model list (Llama 3.1, Mixtral, Falcon) reads like a 2023–2024 snapshot. There's no sign of active development toward the models people actually want to run in mid-2026.

**The model size threshold keeps moving.** When Petals launched, running a 176B model at all was remarkable. Now you can run a 27B model on a phone ([[Bonsai 27B]]), a 70B on a gaming GPU ([[Choosing a GGUF Model]]), and DeepSeek V4 at 96GB with asymmetric quantization ([[DS4 (DwarfStar 4)]]). The models Petals distributes across dozens of peers are increasingly runnable on a single machine. The technology isn't obsolete — the 405B models still need it — but the addressable market shrinks each year as local inference improves.

**The architecture is genuinely elegant.** Unlike AirLLM's temporal decomposition (streaming layers from disk one at a time), Petals uses spatial decomposition across machines. Each approach solves the same problem — "model doesn't fit on my GPU" — from opposite directions. AirLLM buys model scale with time ([[AirLLM]]); Petals buys it with network coordination. Both are fragile, but Petals' fragility is social (will peers show up?) while AirLLM's is mechanical (will the disk be fast enough?).

**The hidden gem: model introspection.** The claim that you can "see hidden states" is undersold. For interpretability researchers, access to intermediate activations is the whole game. A P2P network that lets anyone probe a 405B model's internal representations is a research infrastructure play, not just a cheap-inference play. This is the use case that survives even as single-machine inference improves.

**Why didn't this take off?** The project has a Discord, a chatbot endpoint, a Colab notebook — all the right artifacts for a community. But it never reached the critical mass where you could reliably depend on peers being available. Distributed systems have a network-effect problem: they need enough participants to be useful *before* they're useful enough to attract participants. Petals may have simply been too early, launching when consumer GPUs were scarce and model sizes were at their peak intimidation. The irony is that now, with millions of developers running local models and looking for more capacity, the infrastructure Petals pioneered might actually have the user base to sustain itself — but the project appears dormant.

## Comparative Landscape

| Approach | Mechanism | Bottleneck | Best For |
|----------|-----------|------------|----------|
| **Petals** | P2P network, spatial split | Peer availability, latency | Research, model introspection |
| **AirLLM** | Disk streaming, temporal split | Disk I/O | Single-machine max scale |
| **DS4** | Asymmetric quantization | RAM capacity | Single-machine production |
| **GGUF / llama.cpp** | Uniform quantization | Quality loss at low bits | Interactive chat, broad support |
| **vLLM** | PagedAttention, batching | VRAM (full model) | High-throughput serving |

Each approach optimizes a different constraint. Petals is the only one that sacrifices reliability for radical access — if you can find peers, you can run anything. But "if" is doing a lot of work there.

---
*Sources: [[raw/petals]]*
*Last updated: 2026-07-25*
