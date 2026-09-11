# AuK

Tencent's 1.5B-parameter open-source (MIT) foundation model that collapses speech generation, editing, enhancement, and separation into a single natural-language instruction interface — rather than a shelf of separate TTS, editing, and restoration tools. Released September 2026 with a technical report (arXiv:2609.08936), source code and weights, a distilled AuK-Flash variant for 4-step inference, and Day 0 SGLang-Omni serving support.

Per the report, it is trained on ~3.03 billion instruction–audio instances (1.95M hours of supervision) across five task families, conditions on a multimodal LLM (Qwen2.5-Omni-3B) for semantics and a jointly-trained VAE for acoustics, and generates with a hybrid rectified-flow Transformer (dual-stream MMDiT → single-stream DiT). AuK-Flash does 4-step inference without classifier-free guidance at a 4.5× wall-clock speedup.

---

## Key Quotes

> "AuK is a 1.5B foundation model for speech generation and editing. Trained on millions of hours of diverse audio data, AuK supports zero-shot and instruction-based TTS, content and acoustic editing, paralinguistic editing, speech enhancement, and source separation through a unified natural-language instruction interface."

This is the whole bet in one sentence: one model, one interface, every speech task. The field has spent a decade building separate models for TTS, voice conversion, separation, and enhancement. AuK collapses them into a single checkpoint you address with natural language — the same consolidation move [[SAM Audio]] made for separation, but stretched across the entire speech stack rather than one task. SAM Audio unified the *prompting* of one task; AuK claims the interface covers *five* task families spanning generation, content editing, enhancement/separation, paralinguistic editing, and acoustic editing. That is a "Segment Anything in audio" ambition, extended from "isolate this" to "say this, change this, clean this, re-perform this."

> "AuK exposes every task through the same natural-language instruction interface."

The interface is the product. Instruction-based TTS ("generate speech from a voice description alone — no reference audio"), de-accent, whisper conversion, timbre editing — each is just an instruction, not a separate model or pipeline. This is [[Intent Is the Interface]] applied to audio: the capability list is derived from what you can *ask for*, not from what the architecture was trained to output.

> "a hybrid rectified-flow Transformer that performs dual-stream MMDiT blocks followed by unified single-stream DiT blocks for generation."

The architectural detail worth noticing. MMDiT (multimodal diffusion transformer, the Sora/SD3 lineage) runs two streams so text and audio conditioning can attend to each other before collapsing into a single unified stream for generation. "Rectified flow" is the same flow-matching family [[SAM Audio]] uses — the continuous-diffusion comeback documented in [[Continuous Diffusion Language Models]], here applied to raw audio. AuK is not a new architectural invention; it is a careful assembly of the 2024–2026 generation stack pointed at speech.

> "AuK-Flash performs 4-step inference without classifier-free guidance and achieves a 4.5 wall-clock speedup over the full model under matched conditions."

This is the production reality check. A 4-step sampler that drops classifier-free guidance means you no longer pay for two forward passes per step — the single biggest lever for making a diffusion audio model interactive, and the same few-step-sampling economics that [[Continuous Diffusion Language Models]] identifies as the reason continuous methods returned. The honest "under matched conditions" qualifier is worth respecting: wall-clock speedup, not a quality-matched claim across the board. Every diffusion-audio paper wants to say "real-time"; AuK at least shows the arithmetic.

> "The MLLM encoder and VAE are loaded from separate files at runtime, so missing `text_encoder.*` keys during checkpoint loading are expected."

A small but telling architectural detail from the model card: Tencent doesn't ship its own semantic encoder — it leans on Alibaba's Qwen2.5-Omni-3B as the multimodal frontend. Chinese labs are now composing each other's open models into their stacks, which is exactly the unbundled, swap-compatible future [[Why Open Source Matters for AI]] predicted.

---

## Key Themes

**#concept — The unified speech interface.** Generation, editing, and separation as one model behind one instruction language, rather than a zoo of task-specific checkpoints. The natural-language prompt replaces the modality switch. The five-task-family taxonomy (generation, content, paralinguistic, acoustic, enhancement/separation) is itself a contribution: a map of what "editing audio" actually decomposes into, and a claim that they can be trained jointly rather than as separate heads.

**#tool — Diffusion transformer + borrowed multimodal encoder.** A rectified-flow diffusion transformer in a latent audio space (jointly-trained VAE), conditioned by Qwen2.5-Omni-3B. The generation-and-editing unification is the claim; the diffusion backbone is how it's delivered.

**#pattern — Dual-stream MMDiT → single-stream DiT.** The "condition richly, generate simply" pattern. Multimodal conditioning happens in the dual-stream phase where instruction and audio can cross-attend; generation happens in the unified phase. This split is reusable beyond speech — it is the architectural answer to "how do I inject lots of conditioning without paying for it every step."

**#pattern — Distillation for real-time deployment.** AuK-Flash's 4-step, CFG-free inference is the deployment story, not an afterthought. The full model is a research object; the distilled variant is the thing that could plausibly serve voice products — the same wall [[Pocket TTS]] and [[Inflect-Micro-v2]] attack from the compact-model direction.

**#pattern — MIT-licensed Chinese open weights.** Unlike [[SAM Audio]]'s custom license or most commercial speech APIs, AuK is maximally permissive — a meaningful signal in the [[State of Open Source AI 2026]] landscape where Chinese open weights already route 3× more tokens than US.

**#person — Ziyang Ma et al.** The 30-author Tencent speech team, with Xie Chen and Kai Yu (SJTU) in the author list — the same academic-industrial axis behind much of China's speech ML.

---

## Critical Analysis

**The editing tasks are the real differentiator — and the least benchmarked.** Zero-shot TTS is table stakes in 2026; everyone has it. But *speech content editing* ("rewrite what is said — replace, insert, or remove text"), *lyric editing* (rewrite lyrics while preserving melody and voice), *de-accent*, and *whisper conversion* are genuinely less-crowded capabilities. These are the tasks that matter for post-production workflows — and the model card gives no numbers for any of them; its empty "Performance" and "Model Architecture" headers are a release announcement wearing a model card. The report's abstract asserts "leading performance" on zero-shot generation and instruction-guided editing, also without numbers; the tables live in the paper body. Editing has historically resisted the "one big model" treatment more than generation has, so that is where the evidence needs to land.

**The unified-interface claim is load-bearing and unproven here.** "One model, every task" is a beautiful pitch, but it usually hides a quality tax: a model that does everything is often worse at each thing than a specialist. [[SAM Audio]] made the same bet for separation alone and shipped an eval set and judge model to prove it. AuK ships neither — just a Cookbook with instruction templates. The interesting claim isn't SOTA; it's that generation and editing can share one model. That is the [[Moises — AI Music Separation and Creation]] flywheel thesis (separation data *is* generation data) generalized: enhancement/separation, generation, and editing all exercise the same acoustic representation, so train one foundation and let instructions route the task.

**MIT is the sleeper headline.** Speech foundation models have historically been license-crippled — SAM Audio's custom license, commercial APIs, voice-cloning liability. A 1.5B model under MIT means the weights can be fine-tuned, embedded, and redistributed without a lawyer. That, more than the architecture, is what will get AuK pulled into real products.

**1.5B is the honest middle.** Not [[Inflect-Micro-v2]]'s 9.4M-parameter single-voice compactness, not a commercial API's black-box scale. At 1.5B, AuK is plausibly runnable on a single consumer GPU — locally deployable in a way that distinguishes it from the cloud-only [[MiniMax Models]] speech tiers, while carrying far more capability than the compact TTS models. Inflect is the embedded chip; AuK is the (small) data-center foundation. The two are not competing for the same deployment.

**"Without classifier-free guidance" is the quiet revolution.** CFG is a hack — generate twice (with and without conditioning) and extrapolate — that roughly doubles inference cost. Dropping it via distillation is how diffusion models become interactive. AuK-Flash's 4-step, CFG-free, 4.5× claim is the same move the image and video generation field has been making (e.g., [[SANA-Video 2.0]]'s linear-attention speedups), now arriving in audio.

**Day 0 SGLang-Omni support is a leading indicator.** A serving framework shipping support the day a model drops is the ecosystem voting with code. It's the same signal as Hy3's first-class vLLM/SGLang deployment guides — [[Hy3]] and AuK together sketch Tencent's open-weight platform play, now spanning text/code *and* speech.

---

## Connections

- [[SAM Audio]] — Meta's separation-only foundation model and the closest architectural sibling: same flow-matching DiT family, same open-weights posture. AuK generalizes the "one prompted interface" from separation to generation + editing across five task families, under MIT rather than Meta's custom license, but without SAM Audio's judge model and eval set
- [[Inflect-Micro-v2]] — the 9.4M-parameter fixed-voice TTS is the compact pole; AuK is the foundational pole (reference-voice cloning, editing, separation at 1.5B). Together they bracket the speech-generation design space: tiny-and-specialized vs. general
- [[Moises — AI Music Separation and Creation]] — the productized music-separation counterpart and the separation-to-generation flywheel as a commercial strategy. AuK's music separation and lyric editing are the open-source, API-free, model-level version of the stem workflow Moises sells as a cloud service
- [[Local and Open Source Inference]] — AuK is a data point for the "voice is solved, and now the *unified* voice model is open" version of the thesis, counterweight to the compact single-purpose models that page catalogs
- [[Continuous Diffusion Language Models]] — the flow-matching/rectified-flow family AuK builds on, and the distillability argument behind AuK-Flash
- [[Hy3]] — same lab, different modality. Hy3 is Tencent's text/code MoE; AuK extends the Tencent open-weight footprint into speech, both riding first-class serving support
- [[MiniMax Models]] — the commercial speech API (40 languages, emotional control) that AuK open-sources an MIT alternative to

---

*Sources: [[raw/auk]], [[summary/auk]], [[raw/2609-08936]], [[summary/2609-08936]]*
*Last updated: 2026-09-11*
