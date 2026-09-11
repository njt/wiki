# SAM Audio

Meta's foundation model for prompted audio separation: describe a sound in text, click on it in a video, or mark its timespan, and the model isolates it from the mix. Part of the Segment Anything family (SAM 3 for images/video, SAM 3D for space, SAM Audio for sound). Released December 2025 with paper, code, weights, eval set, and judge model.

---

## Key Quotes

> "A generative separation model that extracts both target and residual stems from an audio mixture using text, visual, or temporal prompts."

The architecture is worth unpacking: this isn't a classifier selecting a frequency band. It's a flow-matching Diffusion Transformer in DAC-VAE latent space — it *generates* the separated audio. The residual stem (everything that isn't the target) gets the same generative treatment. This is philosophically different from classical source separation, which subtracts. SAM Audio imagines both outputs into existence.

> "The first model to introduce span prompting."

Span prompting is the genuinely novel contribution here. Text and visual prompts existed before (e.g., AudioSep, CLAP-based systems). But temporal prompting — "isolate the sound that happens between 6.3 and 7.0 seconds" — is new, and it's the natural interface for audio editing workflows. Producers don't think "isolate the trumpet"; they think "isolate what's happening in bars 12-16."

> "SAM Audio achieves beyond state-of-the-art performance for all prompting capabilities."

The usual Meta claim. The subjective eval scores (3-4.5 range across categories) are solid but what matters more is the judge model release — a trainable evaluator correlated with human ratings. Releasing the judge alongside the model is the honest move. Most papers claim SOTA and leave you guessing whether their eval methodology was tuned to flatter their approach.

---

## Key Themes

- **#concept Prompted audio separation** — the Segment Anything philosophy applied to sound: instead of training a model for each separation task, train one model that takes a prompt. This is the same bet SAM made for images, and it paid off there. Audio is harder (temporal dimension, overlapping sources, less training data), but the architecture choices suggest they learned from SAM's mistakes.

- **#tool Diffusion Transformer in DAC-VAE latent space** — the model generates audio in a compressed latent space using flow matching. This is the same architectural family as Stable Diffusion and recent video generation models. The DAC-VAE (Descript Audio Codec) compresses raw audio into tokens the transformer can work with efficiently. For practitioners: you need a CUDA GPU and the model runs in Python ≥ 3.11.

- **#pattern Multi-modal prompting as the universal interface** — text, visual, temporal, or any combination. The model doesn't care which modality the prompt comes from; all are projected into the same conditioning space. This is the important design pattern: don't build separate models for text-to-separation and image-to-separation. Build one model that eats any prompt.

- **#person Bowen Shi et al.** — the author list is a who's-who of Meta's audio ML team. Shi, Tjandra, Hoffman, Wang, Wu, Richter, Le, Vyas, Chen, Feichtenhofer, Dollár, Hsu, Lee. Meta is consolidating its audio research under the SAM brand, which is smart — the Segment Anything name carries weight after SAM 1 and 2.

---

## Critical Analysis

**The real contribution is the eval infrastructure, not the model.** The model is good but not shocking — diffusion-based audio separation existed before. What's novel: (1) span prompting as a first-class modality, (2) a judge model that makes subjective evaluation reproducible, and (3) an OSS eval set that gives the field a shared benchmark. The model will be surpassed. The eval set and judge might outlive it.

**The "Segment Anything" brand is doing a lot of work.** SAM Audio is closer to "segment some things in audio" than "segment anything" — the domain coverage (general sounds, music, speech) is broad but not exhaustive. Try separating two overlapping conversations in the same frequency range and it'll struggle. This is fine — SAM 1 couldn't segment *anything* either — but the name invites a universality the model doesn't deliver.

**Accessibility is the right use case to lead with.** The Starkey and 2gether-International partnerships aren't just PR. Hearing aids that can isolate a conversation partner's voice in a noisy restaurant are a genuine quality-of-life improvement. The tech isn't there yet (latency, on-device inference), but this is the correct framing for why audio separation matters — not "make your podcast sound better," but "let deaf people participate in conversations."

**Open-weight release with a custom license.** The SAM License (not Apache, not MIT) means you can use the weights but you're in Meta's legal framework. For research this is fine. For commercial deployment you'll want a lawyer. The Hugging Face gated access (request checkpoint, log in with token) is a minor friction that signals Meta is cautious about this one.

**The PE-AV companion model is underplayed.** The blog mentions PE-AV almost as an afterthought, but it's the perceptual backbone that makes multi-modal prompting work — it's what lets the model understand that a click on a trumpet in frame 47 corresponds to a trumpet sound in the audio track. This audiovisual correspondence learning is a harder problem than the separation itself, and the separate paper probably deserves more attention than the SAM Audio blog gives it.

---

## Connections

- [[Local and Open Source Inference]] — SAM Audio is open-weight and locally runnable (Python, CUDA GPU). The model sizes (small/base/large) give a practical quality-vs-compute tradeoff
- [[2025 in LLMs]] — part of the 2025 wave of open model releases, though this is a domain-specific model rather than a general-purpose LLM
- [[A Non-Anthropomorphized View of LLMs]] — Halvar Flake's framing applies here too: this is a function through a latent space, not a proto-ear. The model doesn't "hear"; it maps prompts to separation masks via learned representations
- [[AuK]] — Tencent's 1.5B MIT-licensed 2026 sibling that takes the "one prompted interface" bet further: same rectified-flow DiT family and open-weights posture, but a single foundation spanning separation *and* generation *and* editing across five task families. Speech enhancement, source/music separation, and target speaker extraction are just a few tasks behind one natural-language interface, alongside TTS and editing. SAM Audio unified the prompting of one task; AuK claims the interface scales to the whole speech stack — but ships without SAM Audio's judge model and eval set

---

*Sources: [[summary/sam-audio]], [arXiv:2512.18099](https://arxiv.org/abs/2512.18099), [github.com/facebookresearch/sam-audio](https://github.com/facebookresearch/sam-audio)*
*Last updated: 2026-05-14*
