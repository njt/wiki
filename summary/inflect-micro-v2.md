---
url: https://huggingface.co/owensong/Inflect-Micro-v2
title: "Inflect-Micro-v2"
author: Owen Song
date_fetched: 2026-07-29
date_published: 2026
---

Inflect-Micro-v2 is a local text-to-speech system by independent developer Owen Song. It delivers end-to-end text-to-waveform synthesis in under 10 million parameters (37.5 MB FP32), outputting 24 kHz mono audio with a single fixed English male voice. It runs on CPU and CUDA, with an ONNX Runtime variant also available. License: Apache-2.0.

The architecture is VITS-family: an eSpeak-ng phoneme frontend feeds a monotonic-alignment duration predictor, stochastic latent synthesis with residual coupling flow, and an integrated alias-reduced neural waveform decoder. Long text is split at punctuation boundaries, synthesized in chunks, and joined with edge fades. Controls include speed (0.5–2.0), variation (0.0–1.0), and a deterministic seed for reproducible output.

In a community blind-preference study, Inflect-Micro-v2 achieved a 66.2% preference rate (21 wins, 10 losses, 3 ties). Its UTMOS22 predicted-naturalness score is 4.395. Multi-ASR semantic word error rate averages 3.99% (Qwen3-ASR + Nemotron 3.5), and CPU inference runs at 6.28× real-time. It was benchmarked against KittenTTS Nano, Piper Low, and Supertonic 3 — the developer notes no single metric proves overall superiority.

The voice is synthetic, not derived from a real speaker's recordings. Exact-text exclusion was checked against 87,362 training transcripts. An experimental fine-tuning toolkit allows replacing the built-in voice or adapting to a new language, though quality is dataset-dependent and described as experimental.

Stated limitations: English only, no voice cloning, unfamiliar phrasing can degrade quality, and the model is not validated for medical, legal, emergency, or accessibility-critical use. The developer prohibits impersonation or deception and requires synthetic-speech disclosure where context could mislead.
