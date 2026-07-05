---
title: "maclocal-api"
url: https://github.com/scouzi1966/maclocal-api
date_fetched: 2026-05-14
section: "LLMs"
---

# AFM (macOS Local API)

AFM is a Swift-based command-line tool that exposes Apple's Foundation Models and open-source MLX models through an OpenAI-compatible API endpoint. Runs entirely locally on Apple Silicon Macs without cloud services or API keys.

## Key Features

- OpenAI API Compatible interface for seamless integration with existing tools
- Runs any Hugging Face MLX model locally including Qwen, Gemma, Llama, and 28+ tested variants
- Built-in API Gateway that auto-discovers and proxies Ollama, LM Studio, and Jan
- Privacy-first: all processing happens locally on your device
- Integrated WebUI chat interface with model selection
- Apple Vision OCR capabilities via CLI and HTTP endpoints

## Advanced Technical Features

- Seven+ tool-call formats with auto-detection (JSON, XML, GLM4, Gemma, others)
- Streaming tool-call deltas with token-level start/end detection
- Strict JSON schema enforcement with xgrammar EBNF support
- Radix-tree prefix KV cache for reusing context across turns
- 4/8-bit KV quantization to reduce memory consumption
- Concurrent batch decoding with fair request queueing
- Prometheus metrics endpoint with vLLM-style monitoring
- Retry-After header support for agent coordination

## System Requirements

- macOS 26 (Tahoe) or later
- Apple Silicon Mac (M1/M2/M3/M4)
- Apple Intelligence enabled in System Settings
- Xcode 26 for building from source

## Installation

Three deployment options: Homebrew (`brew install scouzi1966/afm/afm`), pip (`pip install macafm`), or build from source.

## API Endpoints

- POST /v1/chat/completions - OpenAI-compatible chat
- GET /v1/models - List available models
- POST /v1/vision/ocr - Apple Vision OCR (images/PDFs)
- POST /v1/audio/transcriptions - Speech transcription
- POST /v1/audio/speech - Text-to-speech
- GET /health - Server status

## Architecture

Written primarily in Swift (65.7%) with Shell (22.9%) and Python (10.6%). Uses Vapor web framework. Compatible with agentic coding assistants including OpenCode, OpenClaw, Cline, Continue.dev, Aider, Cursor, and Hermes.

License: MIT
