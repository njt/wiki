---
url: https://huggingface.co/jialinyyzz/humanizer
title: "humanizer (jialinyyzz)"
author: jialinyyzz
date_fetched: 2026-10-24
date_published: 2026-10-04
topics:
  - local-and-open-source-inference
  - ai-research-and-models
---

A model card for **humanizer**, a 12B fine-tune of google/gemma-4-12B (Apache 2.0) whose one job is rewriting AI-written drafts — emails, essays, reports, forum posts, in English and Chinese — so they read like a person wrote them, running entirely locally. The card is unusually rigorous for a community model: per-file quantisation tables with fact-check and detector numbers, a byte-exact prompt format, and a full training disclosure.

Headline evaluation: at Originality.ai's strictest setting, 95% of the 210 English rewrites were judged human (11 flagged, down from 26 in v1), with **no AI detector used anywhere in training** — the detector is an external check only. Fact fidelity is measured by a deliberately strict LLM judge (GLM-5.3): 376 of 420 English rewrites had no factual problem, and over 9 in 10 of the flagged fixes are a single word or phrase. Chinese trails English (149 of 204 clean). The model ships as GGUF files from 2-bit QAT (3.9 GB, runs on 8 GB machines) to bf16 (23.8 GB), with quantisation-aware training pushing a same-size 2-bit build from 70% to 87% top-1 agreement with bf16.

Training is disclosed in full: 28,598 SFT pairs of AI draft → real human original (the human side always real human writing), 3,918 DPO pairs chosen only on fact fidelity and copying, then three rounds of GRPO (500 steps) with a fact-judge reward plus a copying penalty. Every training row is published as `jialinyyzz/humanizer-data`. Usage is text-completion, not chat: one fixed instruction prompt, temperature 1.0 / top-p 0.95, stop on EOS only, and byte-for-byte prompt reproduction is specified down to a sha256 prefix. Runtimes covered include llama.cpp, MLX, transformers, vLLM, and a carefully-worded Ollama Modelfile. Notably, the card includes an AGENTS.md so coding agents can set it up and self-test.

Limitations are stated plainly: read the result before sending, especially numbers, dates and names; templated genres (emoji social posts, policy memos) remain hardest for detectors; formatting can be dropped; and detector results "are one measurement on one date, not a promise."
