---
url: https://platform.minimax.io/docs/guides/models-intro
title: "MiniMax Models — Overview"
author: MiniMax
date_fetched: 2026-05-15
date_published: unknown
topics:
  - ai-research-and-models
---

# MiniMax Models — Overview

MiniMax's official models overview page from their developer platform documentation.

## Text Models

### MiniMax-M2.7
Flagship model. "Top real-world engineering," "Professional office delivery," "Character-rich interaction." Supports interleaved thinking and tool use. Deployable locally via vLLM, SGLang (Linux GPU), or MLX (Mac Studio).

### MiniMax-M2.7-highspeed
Same performance as M2.7 with significantly faster inference. "Polyglot code mastery," "Precision code refactoring," "Low latency."

### MiniMax-M2.5
"Optimized for code generation and refactoring." "Peak Performance. Ultimate Value."

### MiniMax-M2.5-highspeed
Same as M2.5 with faster inference. Polyglot code, refactoring, low latency.

### M2-her
Text dialogue model for role-playing and multi-turn conversations. Character customization, emotional expression, multi-turn dialogue.

### Legacy
- MiniMax-M2.1: 230B total, 10B activated per inference. Code gen and refactoring. (Also has highspeed variant)
- MiniMax-M2: Context 200k tokens, output 128k tokens (including CoT). Agentic capabilities, function calling, streaming.

## Audio Models
All support 7 emotions + specified languages/dialects.

| Model | Languages | Notes |
|---|---|---|
| speech-2.8-hd | 40 | Ultra-realistic with sound tags |
| speech-2.8-turbo | 40 | Speed + natural flow |
| speech-2.6-hd | 40 | Ultimate Similarity |
| speech-2.6-turbo | 40 | Value + low latency |
| speech-02-hd | 24 | Stronger replication |
| speech-02-turbo | 24 | Rhythm and stability |

WebSocket TTS handles up to 10,000 chars/request. Async TTS up to 1M chars.

## Video Models
All 24fps.

| Model | Input | Resolutions |
|---|---|---|
| Hailuo 2.3 | T2V, I2V | 1080p 6s; 768p 6s,10s |
| Hailuo 2.3Fast | I2V only | 1080p 6s; 768p 6s,10s |
| Hailuo 02 | T2V, I2V | 1080p 6s; 768p 6s,10s; 512p 6s,10s |

Hailuo 2.3 and 02 claim "SOTA instruction following" and "Extreme physics mastery."

## Music Models
- Music-2.6: "Cover Reborn. Bass Redefined."
- Music-Cover: Cover generation from reference audio (one-step, two-step with lyrics mod, style transfer, auto lyrics extraction)
- Music-2.0: Text to Music. Human-like performance, rich emotional expression.

## API Compatibility
Three compatibility layers: Native Text Generation, Compatible Anthropic API, Compatible OpenAI API. Prompt caching available via Anthropic API-compatible cache_control. MCP support with Token Plan MCP providing web_search and understand_image tools.

## Integrations
Documented setup for Claude Code, Cursor, Windsurf, Cline, TRAE, Hermes Agent, OpenClaw, Mini-Agent, Eigent, and mmx-cli.
