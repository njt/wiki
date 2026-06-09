---
url: https://localai.io/
title: LocalAI
author: Ettore Di Giacinto
date_fetched: 2026-06-09
date_published: 2026-06-02
---

# LocalAI

**The free, OpenAI and Anthropic alternative. A small, composable AI stack.** Run powerful language models, autonomous agents, and document intelligence locally on your hardware. A lean core that pulls model backends on demand, so you install only what you use. No cloud, no limits, no compromise.

Open-source under MIT license. 40k+ GitHub stars. Created by Ettore Di Giacinto (mudler).

## The Local Stack

LocalAI is actually three projects:

1. **LocalAI** — Core LLM/API engine. Drop-in replacement REST API compatible with OpenAI, Anthropic, and ElevenLabs API specifications.
2. **LocalAGI** — Autonomous AI agent management platform. Drop-in replacement for OpenAI's Responses API. Build and deploy autonomous agents with no coding required.
3. **LocalRecall** — REST API for persistent memory, semantic search, and knowledge base management. Perfect for AI applications that need long-term memory.

## Architecture

- **Small core binary** with backends pulled on demand as isolated gRPC services (OCI images). You install only what you use.
- **36+ backends**: llama.cpp, vLLM, transformers, whisper.cpp, diffusers, MLX, and many more.
- **Hardware support**: NVIDIA, AMD, Intel, Apple Silicon (Metal), Vulkan, or CPU-only. No GPU required.
- **Multi-user**: API key authentication, user quotas, role-based access control.
- **Built-in web UI** for chat, model management, and monitoring.

## Key Features

- **LLM Inferencing**: Run LLMs, generate images, audio, and more locally on consumer-grade hardware.
- **Agentic-first**: Extend with LocalAGI for autonomous AI agents that run locally.
- **Memory and Knowledge base**: Extend with LocalRecall for semantic search and memory management.
- **OpenAI Compatible**: Drop-in replacement for OpenAI API. Works with existing applications and libraries.
- **No GPU Required**: Runs on consumer-grade hardware.
- **Multiple Models**: Support for various model families across modalities.
- **Privacy Focused**: No data leaves your machine.
- **Easy Setup**: Docker, binaries, install script, Kubernetes, Podman.
- **Community Driven**: Active Discord, regular updates.

## Quick Start

Docker is recommended:
```bash
docker run -p 8080:8080 --name local-ai -ti localai/localai:latest
```

Install script:
```bash
curl https://localai.io/install.sh | sh
```

Loading models:
```bash
local-ai run llama-3.2-1b-instruct:q4_k_m
local-ai run huggingface://TheBloke/phi-2-GGUF/phi-2.Q8_0.gguf
local-ai run ollama://gemma:2b
```

## Integrations

LangChain integration available via `langchain_community`. Marketplace of compatible apps including Claude Code, AnythingLLM, Dify, n8n, Open WebUI, OpenHands, and more.

## Source Content (from localai.io homepage, fetched via curl)

The site is built with Hugo + Relearn theme. Last modified June 2, 2026.

Homepage emphasizes: composable design (small core, add what you need), drop-in API compatibility (OpenAI/Anthropic/ElevenLabs), no vendor lock-in, local-first privacy.

Navigation structure reveals the full scope: Overview, Installation, Getting Started, News, Features, Integrations, Advanced, References, FAQ.

GitHub: https://github.com/mudler/LocalAI
Discord: https://discord.gg/uJAeKSAGDy
Twitter: https://twitter.com/LocalAI_API
Model catalog: https://models.localai.io
