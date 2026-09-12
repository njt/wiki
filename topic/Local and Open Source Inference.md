# Local and Open Source Inference

Running open-weight models yourself has crossed from a stunt into an ordinary engineering decision, but the honest version of that claim is narrower than the enthusiasm suggests. Open weights now trail the closed frontier by roughly four months on the broadest index and sit within a few points on coding and agentic tasks — yet the gap has been widening since DeepSeek R1, not closing, and on private benchmarks it may be twice as large. What closed the practical distance was published engineering, not faster silicon: mixture-of-experts sparsity, compressed attention and KV caches, multi-token prediction, and aggressive low-bit quantisation — the same techniques that make frontier-scale models runnable on hardware a person actually owns. The constraint that binds is memory, not compute. And the mature practitioners' conclusion has shifted: a local model is a different tool for bounded, private, fixed-cost work, not a worse copy of a frontier model, and it is still unsafe to leave unattended on long horizons. The conviction underneath nearly every source is that a model you can download, quantise, and run is infrastructure you own rather than a service you rent.

---

## The Argument

### Open weights track the frontier closely — and the gap is growing, not shrinking

Epoch AI's Capabilities Index put the average lag of leading open-weight models behind frontier closed models at roughly four months through early 2026 — an 8-point gap the authors compare to the distance between GPT-5 and GPT-5.5, and slightly wider than the ~3-month average of prior years. Håvard Tveit Ihle's 17-benchmark audit on LessWrong splits the same question by benchmark type: open models trail by 4–6 months on public benchmarks and 8–10 months on private ones. Both analyses agree on the direction of travel — the gap was smallest around DeepSeek R1 in January 2025 and has been growing since — and both warn that the published numbers probably understate it, because open-weight labs optimise against public benchmarks and third-party providers serving open models can degrade them subtly. On reasoning-heavy private benchmarks the open models lag most; on coding and agentic tasks they sit within a few points.

The counter-evidence is that the gap no longer matters for the work people actually do. Nathan Lambert's read of GLM-5.2's June 2026 release — MIT-licensed weights landing 204 days after Claude Opus 4.5 — is that it is "the first open model that feels right in coding harnesses as a general agent," a reception he compares only to DeepSeek R1. [[GPU Self-Hosting for Coding Agents]] found Kimi K3 achieved the highest task resolution (86.4%) of any model it tested, but at 8× the latency of the Claude Code API baseline and requiring B300s to fit in memory. And [[A Few Words on DS4]] records antirez, for the first time, relying on a local model for "serious tasks he would normally ask Claude or GPT," describing the local experience as "a lot more B than A" — a lot closer to frontier than to toy.

The reason the practical distance closed is instructive. Matt Coles's mid-2026 survey argues the driver was not bigger hardware but engineering that cut compute and memory per token: sparse attention (DeepSeek's "lightning indexer"), mixture-of-experts routing, latent KV-cache compression, multi-token prediction, and 4-bit quantisation. His point is the pointed one: these are "published papers and merged commits, not trade secrets." "The models being good is nice, but it's the methods staying out in the open that gives the rest of us options."

### Memory, not compute, is the binding constraint

The most consistent finding across the hardware sources is that VRAM, not FLOPs, is the first limit. [[Qwen3.8 27B Hardware Tests]] measures it precisely: a Q4_K_Small Qwen3.8 27B rises from 18 GB of VRAM at 4k context to 34 GB at 256k — a 24 GB card is practical to 64k, a 32 GB card to 128k, and 256k sits beyond any consumer GeForce. The article is deliberately hardware-only and refuses to judge intelligence, which makes its two comparisons the useful ones: Qwen3.8 is a 2.4× speed *regression* against Qwen3.6 at 64k context, and slower than Meta's Muse Glimmer 30B at every length. Newer is not faster.

The reason is arithmetic, and [[Self-Hosted LLMs]] encodes it: tokens-per-second is roughly memory bandwidth divided by model size, times efficiency, quantisation boost, and context impact. Autoregressive generation is memory-bandwidth-bound, not compute-bound. That one formula explains most of what follows: why the RTX 3090 remains the value pick, why a £200 used datacenter V100 with 900 GB/s of HBM2 is a viable ~32 tok/s box ([[Datacenter GPU in a Gaming PC]]), and why two mid-range cards wired together — [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] reports 80+ tok/s on Qwen3.6 27B Q8 with a 230k context — beat a single card that cannot hold the model plus its KV cache.

Mixture-of-experts changes the shape of the constraint. As [[Self-Hosted LLMs]] documents, only active parameters matter for per-token compute, but every expert must live in memory: MoE is "cheap on compute and bandwidth but very heavy on capacity." That is why unified-memory machines — a Mac Studio, an AMD Strix Halo mini-PC, NVIDIA's RTX Spark — keep showing up as the fit for MoE, and why [[Bonsai 27B]] can put a 27B-class model on a phone by quantising the entire network to 1.125 bits per weight while retaining roughly 90% of baseline. The same logic drives [[DS4 (DwarfStar 4)]]'s asymmetric 2-bit recipe: quantise the routed experts — 48 of DeepSeek V4's 49 parameters — to IQ2_XXS/Q2_K while leaving the shared experts, attention, router, and embeddings at full F16, which is what fits 284B parameters into ~96 GB with usable quality.

Where capacity still bites, the ecosystem has answered with offloading. [[FreeToken — Edge-Native MoE Serving Engine]] treats the GPU, CPU, host RAM, and PCIe bus as one elastic platform, streaming expert weights from pinned RAM and splitting each decode step between GPU and CPU according to measured bandwidth; its paper claims a 35B model on a laptop, 284B on a gaming desktop, and the 753B GLM-5.2 on a single workstation GPU. AirLLM takes the same idea to its limit, streaming one transformer layer at a time from disk so that a 70B model runs on a 4 GB GPU, a 405B on 8 GB, and DeepSeek-V3 on ~12 GB — the "run anything, slowly" option, trading bandwidth for capacity and proving the memory ceiling can be routed around entirely.

The irony of the moment is that the models are finally good enough to run at home right as the box to run them on got expensive: PC DRAM more than doubled in price, SSDs roughly doubled, and memory capacity is sold out into the datacenter, with relief not expected before late 2027. The hardware section of the same survey is watching NVIDIA's RTX Spark — a Grace CPU plus Blackwell GPU with up to 128 GB of unified memory — as the next answer to MoE's capacity problem.

### Engines, gateways, and the bet on depth versus breadth

Between the weights and the user sits a thickening infrastructure layer. [[LocalAI]] is the generalist: a MIT-licensed, 40k-star drop-in replacement for the OpenAI, Anthropic, and ElevenLabs APIs, with a small core that pulls any of 36+ backends (llama.cpp, vLLM, transformers, whisper.cpp, MLX and more) on demand, plus separate LocalAGI and LocalRecall projects for agents and memory. [[Lemonade (Local AI Server)]], backed by AMD, does the same job for multimodal work — chat, vision, image generation, speech-to-text, text-to-speech — and its sub-10 MB embeddable binary lets any application bundle a full local AI server without the user ever knowing it is there. [[Petals — Decentralized LLM Inference]] attacks the hardware problem from the opposite direction, splitting a model across a peer-to-peer network ("BitTorrent-style") so consumer machines collectively serve models up to Llama 3.1 405B at roughly 6 tokens/second — usable, but an order of magnitude off a single-machine engine.

On top of the runners sits a speculative-decoding layer. [[mlx-dspark]] brings lossless speculative decoding to Apple Silicon, running the DSpark and DFlash drafter families plus a drafter-free n-gram mode, with a hardware-aware calibration step that tunes draft length to the specific chip — exactness is preserved because the target model verifies every proposed token. Speculative decoding keeps reappearing across the sources as the cheap, quality-preserving speed-up: [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] reports a 0.77 acceptance rate, [[Local Qwen Is Not a Worse Opus]] reports ~93% acceptance lifting throughput from 67 to 130–200 tok/s, and [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]] runs its drafters' KV cache at q4_0 with no observed quality loss.

The most consequential split is strategic. [[DS4 (DwarfStar 4)]] is a 65,000-line C engine hardcoded to a single model — DeepSeek V4 Flash — with hand-tuned quantisation, a first-class disk KV cache, and an explicit correctness-before-speed engineering order, built in about a week with GPT 5.5's assistance. [[A Few Words on DS4]] makes the bet explicit: deep integration with one good model beats shallow support for many, and for local inference "you load what you need depending on the question." [[LocalAI]], [[Lemonade (Local AI Server)]], and the calculator at [[Self-Hosted LLMs]] make the opposite bet — flexibility across hundreds of models, at whatever quality the upstream runners deliver. Both are coherent; they optimise different things.

### Local is a different tool, not a worse Opus

The most important idea to arrive in this evidence set is a reframe. [[Local Qwen Is Not a Worse Opus]] argues that comparing a local Qwen 27B to Claude Opus is a category error: they are "a different tool" for different work. Alex Ellis's version of the evidence is the looping problem — a local model asked what commands to add repeated the same five suggestions verbatim for half an hour, "burning 600W of my electricity," stuck at the edge of its ability and unwilling to ask for help. His conclusion is that local models are good for bounded, privacy-sensitive, fixed-cost tasks — airgapped customer diagnostics, telemetry analysis that caught a customer under-paying 4–5× for over a year, which alone paid for his ~$12,000 RTX 6000 Pro — and not good for long-horizon unsupervised agentic work. "Even our almost 15k USD card couldn't fix that."

The same reframe arrives from two other directions. [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]] insists the right metric is wall-clock time to a correct result, not tokens-per-second: a "slower" model that finishes a task unattended beats a faster one that keeps you in the loop — and reports a Qwen3.8-27B diagnosing and fixing three bugs overnight, unattended, on a budget two-GPU rig, then verified by a different model. [[Honey I Shrunk the Coding Agent]] holds the model constant and varies the scaffold, showing a Qwen3.5-9B in Q4_K_M move from 19.11% to 45.56% on the Aider Polyglot benchmark when the agent around it is redesigned for a small model's behavioural profile — write-guards, thinking budgets, skill cards. The three together establish the local stack as a system of model, quantisation, engine, and harness, where the model alone predicts little.

### What running your own buys: privacy, sovereignty, and the self-hosted tradition

Voice is the cleanest case of local inference already being solved. [[Doing]] transcribes 60 seconds of audio in about 400 ms with NVIDIA's Parakeet model — roughly 150× real-time, on-device, "your voice never leaves your Mac." [[Handy]] offers Whisper and Parakeet V3 backends for free across platforms. The insight worth keeping from [[Doing]] is that the downstream consumer of transcription is now usually an LLM, which can recover intent from slightly imperfect text — so the slightly lower accuracy of local transcription is good enough for the use case that matters. [[Pocket TTS]] shows the same on the output side: a 100M-parameter voice-cloning model that runs on a laptop CPU.

The deeper claim is privacy and control, and it is where this theme meets a broader self-hosted tradition. [[Thunderbolt]] states it as a tagline: "AI You Control: Choose your models. Own your data. Eliminate vendor lock-in," self-hostable via Docker or Kubernetes and air-gappable. [[Local Deep Research]] runs an entire deep-research pipeline locally — Ollama plus SearXNG, per-user AES-256-encrypted databases, and an egress guardrail classifying what may leave the machine — and reports ~95% SimpleQA and 77% xbench-DeepSearch on a single RTX 3090 running Qwen3.6-27B. [[Indexing 669 GB of GoPro Videos with Local ML]] is the same conviction at personal scale: 669 GB of cycling footage indexed entirely on an M1 Max, where unified memory's ability to hold four models at once beat a faster RTX 3060 that could not. The same "your files stay on your own hardware" instinct shows up in projects only tangentially about AI — [[BookOrbit]] and [[Grimmory]] are self-hosted reading servers, and [[clawdBot]] is a personal assistant that runs locally and delegates through chat apps.

Local coding agents are now a shipping category. [[Magnitude]] installs a terminal agent that profiles your hardware, downloads a model, and runs it in-process with no server and no API keys; [[OpenMono Agent]] and [[OpenMonoAgent]] pair a C# agent loop with a Dockerised llama.cpp server, "unlimited tokens forever," with doom-loop detection and a permission engine. These are the secondary-topic edge of the theme — coding agents that happen to run local — but they are evidence that the local stack has moved from "can I?" to "how do I ship it?".

---

## Where the Sources Disagree

**How far behind, really?** Epoch AI says four months on the Capabilities Index; Ihle's LessWrong audit says eight to ten months on private benchmarks. Both are backward-looking and both suspect the truth is worse than the public numbers — open models overfit public benchmarks, and third-party serving can degrade them. The resolution is that the gap is a property of the benchmark type, not a single number: on coding and agentic work the sources agree open models are within a few points, while on hard reasoning and novel math the closed frontier stays ahead. The enthusiasts (antirez, Lambert on GLM-5.2) are describing the first; the sceptics are measuring the second.

**Can you leave a local model unattended?** This is the sharpest genuine tension. [[Local Qwen Is Not a Worse Opus]] says never — the looping problem made even a $15k card unsafe for long-horizon work. [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]] reports exactly the opposite: an overnight, unattended, three-bug fix. The most convincing reading is [[Honey I Shrunk the Coding Agent]]'s — the difference is the harness, the quantisation, and the task scoping around the model, not the weights alone — but no source has established that experimentally, and the two reports sit in direct contradiction.

**Single-model depth versus multi-model breadth.** [[DS4 (DwarfStar 4)]] hardcodes one model and hand-tunes one quantisation recipe, and by the accounts of its own author it delivers quality the generic runners do not claim to match — but it is locked to DeepSeek V4 Flash or whatever antirez ports next. [[LocalAI]] and [[Lemonade (Local AI Server)]] trade that quality for flexibility. Neither is wrong; a coding agent that needs consistency wants the first, an application that needs "a model, any model" wants the second.

---

## What's Missing

**A clean private-benchmark figure.** Both gap analyses concede that public scores flatter open models, but neither produces a trustworthy private-benchmark number — the honest answer to "how far behind are open models?" is still unknown and probably larger than the ~4-month public figure.

**End-to-end local agent benchmarks.** [[GPU Self-Hosting for Coding Agents]] benchmarks coding agents on SWEBench Pro across hardware tiers, but no source benchmarks a fully local pipeline — voice to transcription to search to generation — against the equivalent cloud pipeline on real tasks. Claims about local quality remain disaggregated: a pipeline is only as strong as its weakest component.

**Multi-model orchestration on one box.** Running transcription, embedding, generation, and safety models simultaneously on a single machine means arbitrating GPU and CPU across competing models, and none of the sources addresses it. It is the problem that will matter most once the individual models are all good enough.

**Windows and Linux parity.** The high-performance path is Apple-first: DS4's Metal backend, [[Doing]], and the unified-memory advantage documented by [[Indexing 669 GB of GoPro Videos with Local ML]] all lean on Apple Silicon. Whether Intel, AMD, and Qualcomm close that gap is unproven.

**Sustainability of single-developer engines.** [[DS4 (DwarfStar 4)]] was built in a week by one person; AirLLM, [[Indexing 669 GB of GoPro Videos with Local ML]], and [[mlx-dspark]] are similarly small teams. The quality is remarkable, but the models they target will be superseded, and the single-model-deep-integration approach that currently delivers the best quality is the one with the least institutional backing behind it.

---

## Also on This Theme

- [[Shieldstral]] — Mistral's 3B safety classifier running on a single GPU, matching guard models many times its size.
- [[Inflect-Micro-v2]] — a 9.4M-parameter CPU text-to-speech model.
- [[AuK]] — an open speech foundation model unifying generation and editing behind one instruction interface.
- [[Dolphin]] — ByteDance's 3B document-intelligence model for digital and photographed documents.
- [[Capybara]] — ByteDance's unified text-to-image, text-to-video, and editing model.
- [[maclocal-api]] — an Apple Silicon gateway exposing Foundation and MLX models through an OpenAI-compatible API.
- [[QMD]] — local document search over small GGUF models, with an MCP server so LLM clients can search without uploading.
- [[tolaria]] — a files-first, git-first knowledge base.
- [[Upwelling]] — Git-style branching and merging for collaborative writing.
- [[MimiClaw]] — a ReAct agent loop on a $5 ESP32-S3 microcontroller.
- [[Gemma Gem]] — Gemma 4 running in a Chrome tab via WebGPU.
- [[Smolbox]] — a Claude-Code-like environment compiled to WASM in one browser tab.
- [[AI Brain for Flipper]] — AI control over Flipper Zero hardware through OpenRouter.
- [[Talon]] — hands-free computer control with local speech processing.
- [[SuperStation One]] — an open-source FPGA PlayStation recreation.
- [[Introduction to Obsidian]] — a "file over app" field report.
- [[Headscale]] — the self-hosted Tailscale control server.
- [[Locker]] — self-hostable file storage with a plugin system.
- [[Claude's System Prompt]] — a leaked cloud model's safety architecture, the thing local inference is trying to replicate or diverge from.
- [[RedGridLink]] — offline team navigation over a BLE mesh.

---

*Compiled from 34 sources: [[summary/2026-08-17-qwen3-8-27b-wall-clock]], [[summary/2608-16157]], [[summary/airllm]], [[summary/antirez-ds4]], [[summary/bonsai-27b]], [[summary/bookorbit-app]], [[summary/clawdbot]], [[summary/doing]], [[summary/ds4]], [[summary/freetoken]], [[summary/glm-52-is-the-step-change-for-open]], [[summary/gpu-self-hosting]], [[summary/grimmory]], [[summary/handy]], [[summary/honey-i-shrunk-the-coding-agent]], [[summary/how-far-behind-are-open-models]], [[summary/iliashaddad-gopro-video-indexing]], [[summary/imil-rtx-5080-rtx-3090-qwen-setup]], [[summary/lemonade-server]], [[summary/local-ai-is-not-opus]], [[summary/local-deep-research]], [[summary/local-models-mid-2026]], [[summary/localai]], [[summary/magnitude]], [[summary/mlx-dspark]], [[summary/open-closed-eci-gap]], [[summary/openmono-agent]], [[summary/openmonoagent]], [[summary/petals]], [[summary/pocket-tts]], [[summary/qwen3-8-27b-hardware-tests]], [[summary/self-hosted-llms]], [[summary/thunderbolt]], [[summary/v100-local-llm]]*
*Last compiled: 2026-09-12*
