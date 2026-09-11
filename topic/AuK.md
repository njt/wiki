# AuK

Tencent's 1.5B-parameter open-source (MIT) foundation model that folds speech generation, editing, enhancement, and separation into a single natural-language instruction interface. Released September 2026 with a technical report (arXiv:2609.08936), a distilled AuK-Flash variant for 4-step inference, and Day 0 SGLang-Omni serving support.

---

## Key Quotes

> "AuK is a 1.5B foundation model for speech generation and editing. Trained on millions of hours of diverse audio data, AuK supports zero-shot and instruction-based TTS, content and acoustic editing, paralinguistic editing, speech enhancement, and source separation through a unified natural-language instruction interface."

This is the whole bet in one sentence: one model, one interface, every speech task. The field has spent a decade building separate models for TTS, voice conversion, separation, and enhancement. AuK collapses them into a single checkpoint you address with natural language — the same consolidation move [[SAM Audio]] made for separation, but stretched across the entire speech stack rather than one task.

> "AuK exposes every task through the same natural-language instruction interface."

The interface is the product. Instruction-based TTS ("generate speech from a voice description alone — no reference audio"), de-accent, whisper conversion, timbre editing — each is just an instruction, not a separate model or pipeline. This is [[Intent Is the Interface]] applied to audio: the capability list is derived from what you can *ask for*, not from what the architecture was trained to output.

> "AuK-Flash | Distilled model for fast 4-step inference"

The diffusion-transformer checkpoint has a distilled sibling that does in 4 steps what the base model does in more. Four-step inference is the difference between a research demo and something you can put in a real-time product — the same few-step-sampling economics that [[Continuous Diffusion Language Models]] identifies as the reason continuous methods returned.

> "The MLLM encoder and VAE are loaded from separate files at runtime, so missing `text_encoder.*` keys during checkpoint loading are expected."

A small but telling architectural detail: Tencent doesn't ship its own semantic encoder — it leans on Alibaba's Qwen2.5-Omni-3B as the multimodal frontend. Chinese labs are now composing each other's open models into their stacks, which is exactly the unbundled, swap-compatible future [[Why Open Source Matters for AI]] predicted.

---

## Key Themes

**#concept — The unified speech interface.** Generation, editing, and separation as one model behind one instruction language, rather than a zoo of task-specific checkpoints. The natural-language prompt replaces the modality switch.

**#tool — Diffusion transformer + borrowed multimodal encoder.** A diffusion transformer in a latent audio space (VAE), conditioned by Qwen2.5-Omni-3B. The generation-and-editing unification is the claim; the diffusion backbone is how it's delivered.

**#pattern — Distillation for real-time deployment.** AuK-Flash's 4-step inference is the deployment story, not an afterthought. The base model proves quality; the distilled variant is what actually runs.

**#pattern — MIT-licensed Chinese open weights.** Unlike [[SAM Audio]]'s custom license or most commercial speech APIs, AuK is maximally permissive — a meaningful signal in the [[State of Open Source AI 2026]] landscape where Chinese open weights already route 3× more tokens than US.

**#person — Ziyang Ma et al.** The 30-author Tencent speech team, with Xie Chen and Kai Yu (SJTU) in the author list — the same academic-industrial axis behind much of China's speech ML.

---

## Critical Analysis

**The editing tasks are the real differentiator — and the least benchmarked.** Zero-shot TTS is table stakes in 2026; everyone has it. But *speech content editing* ("rewrite what is said — replace, insert, or remove text"), *lyric editing* (rewrite lyrics while preserving melody and voice), *de-accent*, and *whisper conversion* are genuinely less-crowded capabilities. These are the tasks that matter for post-production workflows — and the model card gives no numbers for any of them. The empty "Performance" and "Model Architecture" headers are a telling gap: this is a release announcement wearing a model card. The actual evidence lives in the arXiv technical report, not here.

**The unified-interface claim is load-bearing and unproven here.** "One model, every task" is a beautiful pitch, but it usually hides a quality tax: a model that does everything is often worse at each thing than a specialist. [[SAM Audio]] made the same bet for separation alone and shipped an eval set and judge model to prove it. AuK ships neither — just a Cookbook with instruction templates. Until the technical report's numbers land, "unified" is a design choice, not a demonstrated win.

**MIT is the sleeper headline.** Speech foundation models have historically been license-crippled — SAM Audio's custom license, commercial APIs, voice-cloning liability. A 1.5B model under MIT means the weights can be fine-tuned, embedded, and redistributed without a lawyer. That, more than the architecture, is what will get AuK pulled into real products.

**1.5B is the honest middle.** Not [[Inflect-Micro-v2]]'s 9.4M-parameter single-voice compactness, not a commercial API's black-box scale. At 1.5B, AuK is plausibly runnable on a single consumer GPU — locally deployable in a way that distinguishes it from the cloud-only [[MiniMax Models]] speech tiers, while carrying far more capability than the compact TTS models.

**Day 0 SGLang-Omni support is a leading indicator.** A serving framework shipping support the day a model drops is the ecosystem voting with code. It's the same signal as Hy3's first-class vLLM/SGLang deployment guides — [[Hy3]] and AuK together sketch Tencent's open-weight platform play, now spanning text/code *and* speech.

---

## Connections

- [[SAM Audio]] — Meta's separation-only foundation model. AuK is the same "one prompted model" philosophy extended from separation to generation + editing, but under MIT rather than Meta's custom license, and without SAM Audio's judge model and eval set
- [[Inflect-Micro-v2]] — the compact fixed-voice TTS pole. AuK is the foundation-model pole: reference-voice cloning, editing, and separation at 1.5B params vs. one English voice at 9.4M
- [[Moises — AI Music Separation and Creation]] — the productized music-separation counterpart. AuK's music separation and lyric editing are the open-source, API-free version of the stem workflow Moises sells as a cloud service
- [[Hy3]] — same lab, different modality. Hy3 is Tencent's text/code MoE; AuK extends the Tencent open-weight footprint into speech, both riding first-class serving support
- [[MiniMax Models]] — the commercial speech API (40 languages, emotional control) that AuK open-sources an MIT alternative to

---

*Sources: [[raw/auk]], [[summary/auk]]*
*Last updated: 2026-09-11*
