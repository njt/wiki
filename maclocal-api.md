# maclocal-api

A Swift CLI that exposes Apple's Foundation Models and open-source MLX models through an OpenAI-compatible API endpoint, running entirely on-device on Apple Silicon. No cloud, no API keys, no privacy leaks. Also aggregates local inference servers (Ollama, LM Studio, Jan) behind one gateway, so you can point any OpenAI-compatible client at a single URL and let it route.

---

## Key Quotes

> "All processing happens locally on your device"

> Supports seven+ tool-call formats with auto-detection, streaming tool-call deltas, strict JSON schema enforcement with xgrammar EBNF, and radix-tree prefix KV cache for reusing context across turns.

## Key Themes

#local-inference #apple-silicon #openai-compatible #privacy #tool-calling

The interesting part isn't "run models locally" -- Ollama does that. It's the aggregation layer: one endpoint that unifies Apple Foundation Models, MLX models from HuggingFace, and whatever other servers you have running. That's genuinely useful for [[Agentic Coding]] setups where you want a local fallback without rewiring every tool.

The tool-calling support is surprisingly deep for a local inference server -- seven format auto-detection, streaming deltas, schema enforcement. Most local tools treat tool calling as an afterthought. The `Retry-After` header for agent coordination is a nice touch that shows someone actually tried running multiple agents against this.

Requires macOS 26 (Tahoe), which limits the audience today but positions it well for the Foundation Models era.

## Critical Analysis

Strong: serious engineering on tool-call support and KV cache optimization. The API gateway mode solving the "I have three local servers" problem is practical. 28 tested MLX models with evaluation reports shows rigor.

Missing: no mention of how it handles context window limits across heterogeneous models behind the gateway. The macOS 26 requirement means this is future-facing -- you can't use it on shipping hardware today (May 2026).

Connects to the broader trend of local inference becoming production-grade rather than a demo curiosity. Compare with [[PiClaw]] for the self-hosted angle, though that's server-side while this is workstation-side.

---
*Sources: [[raw/maclocal-api]]*
*Last updated: 2026-05-14*
