---
url: https://huggingface.co/owensong/Inflect-Micro-v2
title: Inflect-Micro-v2
author: Owen Song
date_fetched: 2026-07-29
date_published: 2026
---

# Inflect-Micro-v2 — Complete Model Card

## Overview

Inflect-Micro-v2 is a **local text-to-speech (TTS) system** delivering "complete local text-to-waveform speech synthesis under 10M parameters." It outputs **24 kHz mono audio** with a single fixed English male voice, supports deterministic seeds for reproducibility, handles long text via punctuation-aware chunking, and runs on both CPU and CUDA.

**Creator:** Owen Song (HuggingFace: [owensong](https://huggingface.co/owensong)), an independent developer who built and funded the project without institutional backing.

## Architecture

Inflect v2 is a **VITS-family** end-to-end text-to-waveform generator using:

- An **English phoneme frontend** (eSpeak-ng based)
- **Monotonic alignment** for duration prediction
- **Stochastic latent synthesis** with residual coupling flow (4 flow blocks)
- An **integrated alias-reduced neural waveform decoder**

### Component Breakdown (Micro-v2)

| Component | Value |
|---|---|
| Latent channels | 192 |
| Text hidden channels | 96 |
| Encoder layers / heads | 3 / 2 |
| Feed-forward channels | 768 |
| Flow coupling blocks | 4 |
| Initial decoder channels | 320 |
| Upsample rates | 8, 8, 2, 2 |
| Training segment | 16,384 samples |
| Output | 24 kHz mono |

There is also a smaller sibling, **Inflect-Nano-v2** (3.97M params, 15.97 MB), which prioritizes minimal footprint over peak quality.

## Parameter Count & Footprint

- **Deployable parameters:** 9,356,513
- **FP32 weight size:** 37.53 MB
- **License:** Apache-2.0

## Capabilities

### Controls

| Control | Default | Range | Effect |
|---|---|---|---|
| `speed` | 1.0 | 0.5–2.0 | Lower = slower |
| `variation` | 0.667 | 0.0–1.0 | Lower = steadier output |
| `seed` | 0 | integer | Reproducible sampling |

### Long Text Handling
Long passages are split at punctuation boundaries into chunks, synthesized separately, and joined with controlled pauses and edge fades. This is not a single unlimited autoregressive pass.

### Output Quality
Demonstrated on held-out text categories including conversational speech, punctuation-heavy text, numbers (spoken-form), names/places, and technical content.

## Supported Runtimes

- **PyTorch** (canonical runtime) — CPU and CUDA
- **ONNX Runtime** — published separately as `Inflect-Micro-v2-ONNX`, supporting CPU/CUDA/DirectML without PyTorch imports, with dynamic lengths and deterministic seeds

## Benchmarks

### 1. Human Blind Preference (Community Study)

Inflect-Micro-v2 achieved a **66.2% preference rate** (21 wins, 10 losses, 3 ties) in an anonymous listening study where systems were hidden and left/right order was randomized. Ties counted as half a win.

### 2. Predicted Naturalness (UTMOS22)

**Score: 4.395** (95% bootstrap CI: 4.381–4.408) based on 500 identical unseen prompts. UTMOS22 is a learned predictor, not human MOS.

### 3. Intelligibility (Multi-ASR Semantic WER)

The headline score is a two-ASR mean (Qwen3-ASR + Nemotron 3.5), excluding Whisper due to insertion-heavy hallucinations on certain competitor clips.

**Micro-v2 two-ASR headline: 3.99%**

**Full three-ASR breakdown:**

| ASR System | WER |
|---|---|
| Qwen3-ASR | 2.52% |
| Nemotron 3.5 | 5.45% |
| Whisper large-v3 | 2.73% |

**Competitor context (two-ASR headline):** KittenTTS Nano voices scored ~2.15–2.39% / ~3.80–3.96%; Piper Low voices scored ~2.62–2.81% / ~5.51–5.60%; Supertonic 3 (3-step) scored 3.03% / 6.04%.

### 4. CPU Runtime (4-thread, 8 vCPU, 32 GB RAM)

- **Inflect-Micro-v2:** 0.1593 RTF → **6.28× real-time**
- **Inflect-Nano-v2:** 0.0933 RTF → **10.72× real-time**

Speed comparator context (50-prompt pass, not perfectly matched): Piper Low ~31.37×, KittenTTS Nano ~13.33×, Supertonic 3 (3-step) ~10.15×.

### 5. Complete Weight Footprint

FP32: **37.53 MB** for Micro-v2; **15.97 MB** for Nano-v2. The integrated waveform decoder is included in both totals.

## Comparison Set

The model was benchmarked against **KittenTTS Nano**, **Piper Low**, and **Supertonic 3** — all compact or local TTS baselines. The developer states that "no single metric is treated as proof of overall superiority."

## Training Data

- Contains **one fixed synthetic English voice** — does not redistribute a real-speaker recording corpus
- The voice is **not** claimed as the identity of a real person
- The training corpus-generation pipeline and private filtering infrastructure are **not** part of the public release
- "Exact-text exclusion was checked against 87,362 training transcripts"
- The release is inference-first, requiring no reference audio or external model at inference

## Adaptation (Fine-Tuning)

An **experimental fixed-voice and language adaptation toolkit** is available at [github.com/owenawsong/Inflect/tree/main/finetune](https://github.com/owenawsong/Inflect/tree/main/finetune). Key constraints:

- A new voice **replaces** the built-in speaker rather than adding runtime voice cloning
- A new language requires owned/licensed speech data, a compatible phoneme frontend, symbol migration, retraining, and fluent-speaker evaluation
- Adapted quality is described as "experimental" and dataset-dependent

## Package Contents

| File/Directory | Purpose |
|---|---|
| `model.pth` | Inference-only generator checkpoint |
| `config.json` | Architecture/audio config (also Hub download-count query) |
| `inference.py` | Public Python API and CLI |
| `inflect_vits_frontend.py` | English normalization, phonemization, punctuation |
| `runtime/` | Self-contained model implementation |
| `samples/` | Held-out example generations |
| `evaluation/final/` | Frozen benchmark prompts, reports, protocol artifacts |
| `docs/` | API, deployment, evaluation, adaptation, export docs |
| `release_manifest.json` | File sizes and SHA-256 hashes |
| `THIRD_PARTY_NOTICES.md` | Third-party component licenses |

## Limitations (Explicitly Stated)

- **English only** with one fixed male voice — not zero-shot voice cloning
- Unfamiliar phrasing can become flatter, less expressive, or less stable
- Numbers, abbreviations, homographs, and uncommon names remain frontend- and context-sensitive
- Long passages use punctuation-aware chunking; transitions can differ from a native long-form model
- Stochastic variation can alter timing and pronunciation (fix the seed for comparisons)
- UTMOS22 and ASR scores do not replace controlled human MOS or MUSHRA-style evaluation
- **Not validated** for medical, legal, emergency, or accessibility-critical communication

## Responsible Use Guidelines

The developer forbids using the included voice to impersonate a real person, deceive listeners, or create fraudulent content. Synthetic speech disclosure is required where context could mislead. Users bear responsibility for applicable laws and license compliance.

## Citation

```bibtex
@software{song2026inflectmicrov2,
  author = {Owen Song},
  title = {Inflect-Micro-v2: Complete Local Text-to-Waveform TTS Under 10M Parameters},
  year = {2026},
  url = {https://huggingface.co/owensong/Inflect-Micro-v2}
}
```

## Community & Contact

- **Discord:** `b111ue` (fastest for informal questions)
- **Community server:** [discord.gg/CVJYedvzvp](https://discord.gg/CVJYedvzvp)
- **Email:** owen.aw.song@gmail.com (preferred for professional inquiries)
- **Downloads (last month):** 645
- **Spaces using this model:** 4 (including an official playground and community deployments)
- **Finetunes:** 1 model; **Quantizations:** 3 models

## Intended Use

Complete local (on-device) text-to-waveform speech synthesis — suitable for applications needing a lightweight, self-contained TTS engine with reproducible output and no cloud dependency.
