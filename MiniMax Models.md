# MiniMax Models

MiniMax's model lineup spans text, speech, video, and music generation, with a compatibility strategy that wraps their native API in Anthropic and OpenAI compatibility layers. The standout signals: M2.7 runs locally via MLX on a Mac Studio, the Hailuo video models claim "SOTA instruction following," and their speech models handle 40 languages with emotional control across six quality tiers.

---

## Key Quotes

> "Beginning the journey of recursive self-improvement." — MiniMax M2.7 tagline

The recursive-self-improvement framing is marketing, not mechanism, but it signals ambition: MiniMax is positioning M2.7 not just as a model but as an agent substrate. The Anthropic API compatibility layer and documented Claude Code/Cursor/Cline integrations confirm they're targeting the coding-agent ecosystem directly.

> "Same performance as M2.7" with "significantly faster inference" — M2.7-highspeed

The highspeed variants across the lineup imply speculative decoding or distillation. MiniMax doesn't disclose the technique, but having a fast/slow split at every tier is a pattern worth watching — it mirrors the turbo-vs-standard dynamic that OpenAI pioneered and everyone else copied.

> Compatible Anthropic API, Compatible OpenAI API

This is the real product strategy: don't compete on API surface, compete on model quality. You can swap MiniMax into any Anthropic or OpenAI SDK pipeline by changing a base URL. For agent frameworks that already target these APIs, the switching cost is near zero.

---

## Key Themes

#model #multimodal #chinese-ai #speech #video

---

## Critical Analysis

The model breadth is impressive on paper — text, speech, video, and music from one provider — but breadth without benchmarks is a catalog, not a comparison. The models page includes zero benchmark scores, no pricing, and only one context window figure (M2's 200k input / 128k output). This is a marketing page, not a technical reference. Compare with Anthropic's model pages, which publish standardized eval tables.

The three-layer API compatibility strategy (native + Anthropic-compatible + OpenAI-compatible) is pragmatically smart. It acknowledges that developers don't want another API to learn. But compatibility layers leak — the page silently drops M2.7 features (tool use, interleaved thinking) when routed through the Anthropic layer versus native. That complexity lives in documentation, not in this landing page.

M2.7's local deployment story via MLX on Mac Studio is genuinely interesting. Most Chinese frontier models don't bother with Apple Silicon support. This puts M2.7 in the same conversation as [[Self-Hosted LLMs]] and [[maclocal-api]]. But without published benchmarks, it's impossible to know where it sits relative to Gemma, Llama, or Qwen at similar sizes.

The Hailuo video models claiming "SOTA instruction following" and "Extreme physics mastery" are bold claims with nothing backing them on the page. ByteDance's [[Capybara]] is the obvious comparison point for Chinese multimodal models — and Capybara publishes papers. MiniMax's video models may be competitive, but from this page alone you can't tell.

The speech models are the most credible part of the lineup: six quality tiers across two generations, 40 languages, emotional control, voice cloning, and voice design. That's a production speech API, not a demo. The 1M-character async TTS limit suggests they've solved long-form audio synthesis at scale.

For coding agents: the [[MinMax Skills]] library (11.8k stars) integrates these media APIs directly into Claude Code and other agent harnesses. That's the pragmatic bridge between the model catalog and the actual developer workflow. The skills library is what makes the models usable; without it, you're reading API docs. With it, you're generating music, speech, and video from inside your coding agent.

See also: [[How to Buy Cheap Claude Tokens in China]] for the broader Chinese AI ecosystem context — MiniMax operates inside the same infrastructure constraints and grey-market dynamics.

---

*Sources: [[raw/minimax-models-intro]]*
*Last updated: 2026-05-15*
