# Local and Open Source Inference

Running models on your own hardware is no longer a curiosity -- it's becoming a practical option for specific use cases. Voice transcription is already there: 150x realtime on Apple Silicon ([[Doing]]), free and open-source across all platforms ([[Handy]]). Document parsing is there: [[Dolphin]] handles any document type with a 3B parameter model. Text-to-speech is there: [[Pocket TTS]] runs voice cloning on CPU at 100M parameters, and [[Inflect-Micro-v2]] pushes the boundary further — 9.4M parameters, 37.5 MB, competitive quality at 6× real-time on CPU. What's not there yet: local inference for the kind of complex reasoning that frontier cloud models provide. The gap between "runs locally" and "as good as Claude/GPT" remains wide for general coding and reasoning tasks, but it's narrowing faster in specialized domains than most people expect.

---

## The Landscape

### Voice: The Solved Problem

**Speech-to-text.** [[Doing]] ($49, Mac-only) uses NVIDIA's Parakeet model to transcribe 60 seconds in ~400ms. No cloud, no subscription. [[Handy]] (21.6k stars, free, cross-platform) uses Whisper or Parakeet V3 with Tauri architecture. Both prove that local voice transcription is production-quality today.

The key insight from [[Doing]]: "LLMs know what you mean, even when your words aren't perfect." When the downstream consumer is an AI, you don't need perfect transcription. This changes the quality bar for the entire voice-to-agent pipeline.

**Text-to-speech.** [[Pocket TTS]] (Kyutai, 100M parameters) runs voice cloning on CPU. No GPU required. If the quality holds up at this size, the economics of voice synthesis change entirely: embedded devices, offline applications, privacy-sensitive deployments all become feasible. Together with [[Handy]], this completes the full local voice pipeline: speak, transcribe, process, synthesize speech back. No cloud round-trip needed.

### Document Intelligence

[[Dolphin]] (ByteDance, ACL 2025) handles digital and photographed documents through a two-stage architecture: classify document type, then apply the right parsing strategy. 89.78 on OmniDocBench with the 3B model. TensorRT-LLM and vLLM acceleration signal production intent.

[[Capybara]] (ByteDance, MIT licensed) unifies text-to-image, text-to-video, and instruction-based editing in one model. FP8 quantization and distributed multi-GPU inference for production use. Both represent ByteDance's growing open-source AI portfolio -- production-grade models at a pace rivaling Meta.

### Local LLM Inference

[[maclocal-api]] exposes Apple Foundation Models and MLX models through an OpenAI-compatible API endpoint on Apple Silicon. The aggregation layer -- one endpoint unifying local servers (Ollama, LM Studio, Jan) -- is the practical contribution. Tool-calling support is surprisingly deep: seven format auto-detection, streaming deltas, schema enforcement. Requires macOS 26 (Tahoe), which limits current adoption.

[[Gemma Gem]] runs Google's Gemma 4 in Chrome via WebGPU. No cloud, no API keys. E2B (~500MB, 4GB VRAM) and E4B (~1.5GB, 6GB VRAM). The trajectory is clear: browser-based AI inference is becoming practical for modest models. The practical question is whether these models are capable enough for useful browser automation -- probably fine for reading pages and answering questions, probably not for complex multi-step workflows.

[[Self-Hosted LLMs]] provides the calculator: map your hardware to LLM performance. The "if I have X, what can I do with it?" framing is exactly right. Most people start with hardware constraints, not model preferences.

[[Bonsai 27B]] pushes the envelope further: a 27B-class Qwen 3.6 derivative at 3.9 GB (1-bit) that fits on a phone, retaining ~90% of baseline quality. The phone constraint is a forcing function — if it runs on an iPhone, it runs comfortably anywhere.

### Local Search and Knowledge

[[QMD]] (by Tobi Lutke) is a mini CLI search engine with hybrid BM25 + vector + LLM re-ranking, all local. Three GGUF models (~2GB total) auto-download. MCP integration lets Claude search local documents without uploading them. The honest caveat: "all local is a great idea, but I end up sending everything upstream to a big model anyway."

[[tolaria]] is an open-source Obsidian alternative designed for AI-agent compatibility: files-first, git-first, offline-first. 10.6k stars. The bet: if your knowledge base is the context your agent operates in, the editor needs to produce files agents can read and write.

### Hardware Frontiers

[[AI Brain for Flipper]] gives a Flipper Zero natural language voice control and AI-powered signal synthesis. The risk classification system (low = auto-execute, medium = diff review, high = explicit confirmation) is the right UX pattern for AI-controlled hardware.

[[MimiClaw]] runs a full ReAct agent loop on a $5 ESP32 -- persistent memory in NVS flash, Telegram interface, cron scheduling. Pure C, 0.5W power consumption. The agent's capabilities are limited by hardware constraints, but 5.4k stars prove the appeal of impossibly small AI.

## Key Tensions

**Privacy vs. capability.** Local inference keeps data private. Cloud inference provides superior capability. Most practitioners resolve this pragmatically: local for voice/documents (where models are good enough), cloud for reasoning (where they're not). But the "good enough" line shifts with every model release.

**Cost vs. convenience.** Local inference has upfront hardware costs but zero marginal cost per query. Cloud APIs have zero upfront cost but per-token pricing. For high-volume applications (continuous transcription, document processing pipelines), local breaks even quickly. For occasional use, cloud is cheaper.

**Specialized vs. general models.** The models that run well locally are specialized: voice (Whisper, Parakeet), documents (Dolphin), embeddings (Gemma). General reasoning still requires frontier cloud models. The interesting question is whether specialized local models composed together can approximate general capability -- a local voice model feeding a local search engine feeding a local summarizer.

**Apple Silicon vs. everything else.** [[maclocal-api]] and [[Doing]] are Mac-focused. [[Handy]] is cross-platform. The Apple Silicon advantage in local inference (unified memory, neural engine) creates a platform divide that may widen as Apple's Foundation Models mature.

## What's Missing

**Benchmarks for local agent workflows.** We have benchmarks for individual model tasks (transcription accuracy, document parsing quality). We don't have benchmarks for end-to-end local agent workflows: how does a fully-local pipeline (voice -> transcription -> search -> generation -> TTS) compare to a cloud pipeline on real tasks?

**Local inference orchestration.** If you're running three local models (transcription, embedding, generation), who manages the GPU/CPU allocation? [[maclocal-api]]'s aggregation layer is a start, but multi-model orchestration on local hardware is an unsolved problem.

**Cost calculators for hybrid setups.** The real-world setup is hybrid: local for some tasks, cloud for others. Nobody has built the calculator that tells you "for your usage pattern, run these models locally and these in the cloud" with cost projections.

**Windows and Linux parity.** The best local inference tools are Mac-first. The Tauri-based tools ([[Handy]], [[tolaria]]) are cross-platform, but the Apple-specific optimizations ([[maclocal-api]], [[Doing]]) don't have Windows/Linux equivalents.

## Key Themes

#local-inference #privacy #voice #document-parsing #hardware #apple-silicon

## Pages

- [[Doing]] — Fast local voice transcription for Mac. $49, no cloud, 150x realtime
- [[Handy]] — Free open-source speech-to-text. Press shortcut, speak, release, pasted
- [[Pocket TTS]] — 100M parameter text-to-speech with voice cloning. Runs on CPU, no GPU
- [[Talon]] — Hands-free computer control: voice, mouth-sound clicks, eye tracking, Python scripts. Patreon-funded, local-first
- [[Gemma Gem]] — Google's Gemma 4 running locally in Chrome via WebGPU. No cloud, no API keys
- [[maclocal-api]] — Apple Silicon local inference: Foundation + MLX models, OpenAI-compatible API
- [[Self-Hosted LLMs]] — Calculator: map your hardware to LLM performance and inference speed
- [[Petals — Decentralized LLM Inference]] — BigScience's peer-to-peer network for running 100B+ models across volunteer GPUs. BitTorrent-style model serving with model introspection support
- [[AI Brain for Flipper]] — Voice-controlled AI for Flipper Zero hardware
- [[RedGridLink]] — Offline MGRS navigation + BLE team sync for 2-8 people. No cell service
- [[Thunderbolt]] — Mozilla/MZLA's open-source cross-platform AI client pivoting to enterprise: sovereign cloud, air-gapped deployments, ACP+MCP protocol support, deepset/Haystack partnership
- [[SuperStation One]] — Affordable FPGA PS1 recreation on MiSTer-compatible Cyclone V hardware. Open-source from day 1 as competitive strategy against Analogue's proprietary premium pricing
- [[Introduction to Obsidian]] — Practitioner's field report on Obsidian: file-over-app philosophy, plugin minimalism, honest graph-view skepticism
- [[Lemonade (Local AI Server)]] — AMD-backed local multimodal AI server: chat, vision, image gen, speech behind an OpenAI-compatible API. Embeddable <10MB binary is the sleeper feature
- [[A Few Words on DS4]] — antirez's DS4 crosses the local inference threshold: he now uses a local model instead of Claude/GPT for serious work. Model-agnostic shell, expert variants, distributed inference ambitions
- [[DS4 (DwarfStar 4)]] — The C inference engine behind DS4: 65K lines, Metal/CUDA, mmap-loaded GGUF, asymmetric quantization, disk KV cache, OpenAI+Anthropic+Responses API, built in 1 week with GPT 5.5
- [[tolaria]] — Open-source Obsidian alternative: files-first, git-first, AI-agent compatible
- [[Upwelling]] — Ink & Switch editor: branching and merging for writers, not just programmers
- [[Claude's System Prompt]] — Leaked Claude Opus 4.6 system prompt, read as a catalog of solved failure modes
- [[Headscale]] — Open-source, self-hosted Tailscale control server. WireGuard mesh networking without the cloud. 38.4k stars
- [[Locker]] — Open-source self-hostable Dropbox/Google Drive alternative: multi-store backends, AI knowledge base, plugin system, virtual bash shell
