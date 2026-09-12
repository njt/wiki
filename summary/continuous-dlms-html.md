---
url: https://sander.ai/2026/08/24/continuous-dlms.html
title: Continuous diffusion language models
author: Sander Dieleman
site: sander.ai
published: 2026-08-24
fetched: 2026-09-04
topics:
  - ai-research-and-models
---

Sander Dieleman (Google DeepMind) surveys the five-year history of **continuous diffusion language models (CDLMs)** — the branch of diffusion research that applies Gaussian-noise corruption to continuous embeddings of discrete tokens, rather than corrupting the tokens themselves (the **discrete diffusion language model / DDLM** approach). It is an insider's update: Dieleman co-authored two of the early 2022 CDLM papers (SED and CDCD) and admits to having been "wistful" when continuous methods went extinct.

The narrative arc: 2021 brought discrete diffusion (multinomial, D3PM, SUNDAE); 2022 brought continuous diffusion (Diffusion-LM, SED, CDCD); late 2023 saw the "continuous extinction," when the field swung almost entirely to discrete methods — Dieleman speculates the ChatGPT moment shifted focus from theoretical elegance to raw performance, and Plaid-1B's measured **64×** training-inefficiency gap made continuous methods hard to take seriously. The tide turned in the second half of 2025 with **hybrid** discrete-continuous methods (diffusion duality, CADD, CCDD, CANDI), then in 2026 with a full resurgence: flow maps extended to the categorical setting (Categorical Flow Maps, Flow Map Language Models, Discrete Flow Maps) followed by a "Cambrian explosion" of CDLM papers (LangFlow, Spherical/Hyperspherical flows, LDLM, ELF, CoBit, RePlaid).

The core technical sections walk the four ingredients every CDLM needs: an **embedding strategy** (explicit one-hot vs. pre-trained vs. jointly learned), a **loss function** (MSE vs. cross-entropy vs. likelihood bounds), a **noise schedule** (crucial because meaningful corruption happens across a narrow band of noise levels, motivating online/adaptive schedules), and **self-conditioning** (passing the denoiser's previous prediction forward — ubiquitous despite breaking the statelessness assumption behind ODE/SDE sampling).

Dieleman's headline thesis: CDLMs are back primarily because of their **distillability advantage**. Few-step and single-step sampling comes naturally to continuous diffusion, whereas discrete diffusion hits a wall — simultaneously sampled tokens are assumed conditionally independent, so a single-step discrete model cannot capture token correlations. The article also flags two open problems: **evaluation methodology** (the quality/diversity trade-off is unquantified, and generative perplexity / GenPPL is easily gamed), and the **semantic gap** between abstract language tokens and low-level perceptual representations that undermines the naive "continuous everywhere = easy multimodality" argument.
