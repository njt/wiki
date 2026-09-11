---
url: https://github.com/Tencent-Hunyuan/AuK
title: AuK — An Open-Source Foundational Model for Speech Generation and Editing
author: Tencent Hunyuan (Ziyang Ma, Zhikang Niu, Wenming Tu, et al.)
date_fetched: 2026-09-11
date_published: 2026-09-09
---

## Source Content

AuK is a 1.5B-parameter open-source (MIT) foundation model for speech generation and editing from Tencent Hunyuan, trained on millions of hours of diverse audio. It ships as two variants: **AuK** (base, high quality, configurable) and **AuK-Flash** (distilled, fixed 4-step inference). A technical report is on arXiv (2609.08936); weights are on Hugging Face and ModelScope.

### Architecture

Three components compose at inference: a **frozen Qwen2.5-Omni-3B** multimodal LLM encodes text/audio instructions (the unused vision tower is dropped); a **BigVGANFlowVAE** (BigVGAN decoder + VITS residual-coupling flow + VAE, 480× downsample, 64-dim latents) converts audio to and from latent space; and a **Flux2Edit** transformer — a Flux2Audio variant with two phases (double-stream MMDiT joint text-audio attention, then single-stream DiT over concatenated text+audio) — runs conditional flow matching over VAE latents with an ODE solver. Only the transformer plus a learned layer-fusion over the LLM's hidden states are trained; the LLM and VAE stay frozen.

### Tasks

Everything goes through one instruction interface (ChatML messages): zero-shot TTS, instruct TTS (no reference audio), content/lyric editing, pitch/speed/volume editing, emotion/timbre/de-accent/nonverbal/whisper editing, speech enhancement, speech/music separation, and target speaker extraction. A reference audio, when present, is VAE-encoded and prepended to the noised latent in the sequence dimension, so "editing" is conditioned prefix-continuation rather than a separate operation.

### Interface & Tooling

Python API (`AukInfer`), a `auk-infer` CLI, a Gradio demo, ComfyUI nodes, and a **Prompt Enhancer** — an LLM-driven front-end (OpenAI-compatible LLM + VAD + ASR) that classifies a free-form request into a task type, normalizes the instruction, estimates target duration, preprocesses audio, and prints a ready-to-run command. A lightweight JSONL fine-tuning pipeline with frame-length dynamic batching, EMA, and per-t validation-loss curves is included.
