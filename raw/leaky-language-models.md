---
url: https://arxiv.org/html/2607.20723v1
title: "Leaky Language Models: Stealing Architecture and Inference Optimizations via Per-Token Timing"
author: "Sadegh Majidi, Niloofar Mireshghallah, Kazem Taram"
date_fetched: 2026-07-25
date_published: 2026-07-22
---

# Leaky Language Models: Stealing Architecture and Inference Optimizations via Per-Token Timing

## Authors & Affiliation
- **Sadegh Majidi** (Purdue University)
- **Niloofar Mireshghallah** (Carnegie Mellon University)
- **Kazem Taram** (Purdue University)

Published on arXiv: 2607.20723v1 [cs.CR], July 22, 2026. License: CC BY-NC-ND 4.0.

---

## Abstract

The paper introduces **LeakyLMs**, a set of attacks demonstrating that "key model and deployment details can be inferred using only token generation timing, even when interacting through remote APIs." Two core attacks are presented: one targeting inference optimizations (detecting speculative decoding and identifying draft model context length), and another recovering architectural properties (number of layers, hidden dimension size, attention heads). The architecture attack builds a detailed timing model for modern NVIDIA GPUs and performs a search over the architecture space. Results show "the near-correct architectural configuration appears in the top-10 guesses more than 90% of the time" for Llama models.

---

## 1. Introduction

The authors frame the work in the context of the global AI race, noting that "model architecture and deployment strategies are valuable assets that provide competitive advantage." Providers keep details of advanced models confidential. The key insight is that providers optimize for low-latency streaming, which exposes "fine-grained, controlled, and accurate per-token generation timings" to external observers.

**First attack** (inference optimizations): Detects speculative decoding by exploiting the fact that draft models typically have shorter context windows than main models. By crafting prompts that require distant context, the attacker induces disagreements between draft and main models, causing measurable timing changes. The attack revealed that "Google Gemini Flash 2.5 uses speculative decoding with a draft context window of approximately 128K tokens."

**Second attack** (architecture): Recovers L (layers), H (hidden dimension), and A (attention heads). The authors first analytically derive asymptotic relationships between runtime and model parameters, then empirically refine with linear regression. The timing predictor is used as an oracle to search over possible configurations.

**Responsible disclosure**: The speculative decoding attack was disclosed to Google on Nov 7, 2025, and acknowledged on Jan 29, 2026.

---

## 2. Background

### 2.1 Transformer Architecture
Decoder-only transformers consist of L decoder blocks plus embedding and projection layers. Five key parameters govern computation: **H** (hidden dimension), **L** (number of layers), **A** (attention heads), **I** (intermediate MLP dimension), and **T** (sequence length). The first four define the architecture; T is user-controlled. The paper considers both eager attention (cuBLAS-based) and FlashAttention2 implementations.

### 2.2 Autoregressive Decoding
Each forward pass generates one token. The model "generates one token per forward pass by computing a conditional probability distribution over the vocabulary." Tokens are streamed as produced.

### 2.3 Key-Value Caching
KV caching splits inference into **Prefill** (processing the prompt and building the cache) and **Decoding** (generating one token per pass using cached representations). Prefill is compute-intensive; decoding is memory bandwidth–bound.

### 2.4 Speculative Decoding
A smaller draft model proposes candidate tokens; the main model verifies them in a single forward pass. "If the outputs of the draft and target models agree, all N tokens are accepted." When they disagree, only one token is accepted, falling back to normal latency. This creates input-dependent timing variation.

---

## 3. Threat Model

The adversary seeks to infer architectural parameters (L, H, A, I) and detection of speculative decoding with draft model context length. The target is a decoder-only transformer accessible via a standard API with streaming. The attacker can only observe token timing—no access to logits, activations, or system logs. No privileged network position is assumed. For the architecture attack, the attacker must know the GPU type used and have access to timing data on the same GPU class for training predictors. The attack assumes single-GPU inference.

---

## 4. Leaking Inference Optimizations

### 4.1 Attack Overview
Speculative decoding is analogous to speculative execution in processors. The key insight: draft models typically have shorter context lengths than main models. "If a prompt requires a context longer than the draft model's maximum context length," the draft model produces incorrect predictions, tokens are rejected, and latency falls back to the large model's timing. By gradually increasing prompt length, a sudden latency spike reveals speculative decoding, and its position reveals the draft model's context length.

### 4.2 Crafting the Prompt
Prompts use a number-memory task: "We have a {n} digit number NUM=" followed by a random number, padding characters, then a query about the number. Padding length is varied to control required context length. Prompt order is randomized to avoid caching or throttling effects.

### 4.3 Detection and Parameter Estimation
**Detection**: A timing spike after a certain input length indicates speculative decoding. **Parameter estimation**: Binary search identifies the breakpoint T_break; draft context length = T_break - prologue_length.

### 4.4 Experimental Setup

#### 4.4.1 Local Experiments
Used Guanaco 13B as main model and TinyLLaMA 1.1B as draft model on an NVIDIA L40 GPU.

#### 4.4.2 Remote Experiments
Tested against Google Gemini, OpenAI GPT, Cohere, and Mistral APIs. Prompt lengths varied up to provider context limits (up to 1M tokens). The client recorded timestamps at each stream event.

### 4.5 Results

#### 4.5.1 Local White-Box
A sharp rise in per-token time was observed at ~2030 tokens. Estimated draft context length: 2048 tokens (matching TinyLLaMA 1.1B's actual context length). Disabling speculative decoding eliminated the spike.

#### 4.5.2 Remote Black-Box
**Gemini Flash 2.5, Flash 2.5 Lite, and Flash 1.5** all exhibited timing spikes indicating speculative decoding. **Parameter estimation**:
- Gemini Flash 2.5: ~128K tokens draft context
- Gemini Flash 2.5 Lite: ~128K tokens
- Gemini Flash 1.5: ~32K tokens

Other APIs showed no detectable pattern, though the authors note this could be false negatives.

---

## 5. Leaking Model Architecture

### 5.1 Attack Overview
A hybrid analytical–empirical approach: theoretical asymptotic analysis is combined with empirical corrections. "We leverage the modular structure of the transformer architecture by decomposing its runtime into primitive operations." Linear regression is deliberately chosen for explainability, generalizability, and configurability.

The predictor serves as an oracle: for each candidate architecture in a search grid, synthesized timing traces are compared to observed traces from the target. Top matches represent the most likely architecture.

Three implementation variants are modeled: Eager (cuBLAS-based), FlashAttention2, and KV-cache-enabled FlashAttention2.

### 5.2 Offline Phase

**Building polynomial features**: Each decoder block is decomposed into subcomponents. For each primitive operation, theoretical compute and memory costs are derived as functions of H, L, A, I, T, and bit-width b. Example: Q-projection of size T×H by H×H costs O(TH²) operations and O(bTH + bH²) memory reads. Runtime is expressed as a linear combination of theoretical terms with learned coefficients.

**Collecting timings**: Timing data is collected from instrumented open-access models (Llama 3.2 1B as reference) across multiple architectural variants. A dataset records architectural properties, input lengths, and per-component timings.

**Composing the predictor**: Separate linear regression models for each subcomponent (attention, MLP, normalization, embedding/projection, overhead). The total runtime is the sum of all components scaled by layer count. For attention: t_att = a₁H²TL/d + a₂T²HL/d + a₃bT²AL/c + ... + C

**Training the runtime corrector**: For Eager attention, cuBLAS dynamically selects kernels based on operand shapes, creating non-linearities. A two-phase ensemble corrects this: (1) a LightGBM classifier predicts which kernel will be selected, (2) random forest regressors predict per-kernel runtime.

**Modeling FlashAttention**: Modified attention terms to reflect FlashAttention2's reduced memory access complexity: Θ(T²D²M⁻¹) where M is per-SM SRAM size. No matmul corrector is needed since FlashAttention doesn't use cuBLAS.

**Modeling KV-Cache**: Separate prefill and decoding terms. Prefill follows Flash2 scaling; decoding reflects single-token processing with attention over cached sequence. Prefill regressors trained on first forward pass only; decoding regressors on remaining passes.

### 5.3 Online Phase

**Building the search space**: Exploits constraints—dimensions are integers, hidden sizes are multiples of 64 or 128, layer counts are typically even, per-head dimensions divide H evenly. Implausible combinations are pruned.

**Reverse engineering**: For each candidate configuration, the predictor generates timing sequences for the same prompts used on the target. RMSE is computed between predicted and observed sequences. Candidates are ranked by distance.

**Scaling for unseen dimensions (Eager)**: The corrector models predict kernels and runtimes for matrix multiplications in unseen configurations, adjusting the final estimate.

**Ranking with KV-Flash2**: Uses a two-stage procedure—first narrow to top-35 using prefill only, then refine with both prefill and decoding predictions.

### 5.4 Experimental Setup
- **Reference model**: Llama 3.2 1B
- **Target model**: Llama 3.2 3B (plus cross-family models: Qwen2.5 1.5B, Phi3.5-mini 3.8B, Gemma2 2B)
- **Hardware**: RTX 2080 Ti (eager), A10 (FlashAttention2), B200 (KV-cache)
- **Search grid**: 1,540 candidate configurations evaluated per target; ~20 minutes on 40 CPU cores
- **Top-5 accuracy**: Success if all targeted parameters are within one step of ground truth (step sizes: L=1, H=128, A=4, I=1024)
- **Remote API**: Weights & Biases Llama 3.1 8B Instruct

### 5.5 Results

**Timing prediction accuracy (NRMSE)**:
| Variant | Train | Test |
|---------|-------|------|
| Eager (Corrected) | 0.106 | 0.119 |
| Flash2 decode | 0.258 | 0.209 |
| KV-Flash2 decode | 0.218 | 0.252 |
| KV-Flash2 prefill | 0.168 | 0.188 |

Eager(Naive) without correction dropped from 0.106 to 0.411 on test data, confirming the corrector's effectiveness.

**Architecture leakage (top-5 accuracy)** for Eager(Corrected) test:
- L only: 86.15%
- H only: 71.54%
- (H, L) jointly: 65.38%

For Flash2 test: L=97.69%, H=100%, (H,L)=54.62%. For KV-Flash2 test: L=83.78%, H=70.95%, (H,L)=45.27%.

**Cross-family generalization** (Eager predictor trained on Llama, tested on other families):
- Qwen2.5 1.5B: L=77.27%, H=100%, (H,L)=50.00%
- Phi3.5-mini 3.8B: L=97.50%, H=97.50%, (H,L)=80.00%
- Gemma2 2B: L=100%, H=77.50%, (H,L)=47.50%

**Remote API results** (Weights & Biases Llama 3.1 8B):
- When inferring H with L known: correct value ranked #1
- When inferring H with L fixed: correct rank #4 (top-5)
- Searching over both: near-match (H=4096, L=34) ranked #8/2068; true config (H=4096, L=32) ranked #28/2068

The paper notes that "the predicted timings closely match the observed measurements from the remote API generation times."

---

## 6. Mitigation

Two strategies are discussed but noted as challenging:

1. **Constant per-token time**: Would require making all generations as slow as the worst case, "potentially increasing latency by several times" and defeating optimization purposes.

2. **Buffering**: Aggregate tokens and transmit at constant intervals. However, this "may be impractical for large models or latency-sensitive applications."

---

## 7. Related Work

The authors survey side-channel attacks on neural networks (timing, cache, power, EM, GPU-level), noting that prior LLM-specific work has focused on fingerprinting models or leaking interaction details. Key comparisons:

- **Carlini et al. (2024)** recovers H and projection weights via logit bias from the API. The authors note this "relies on extra information returned by the API and is more costly" compared to their approach.
- **Performance modeling tools** like Accel-Sim, Amali, and various analytical frameworks are cited, with the observation that most "adopt a system-centric view" lacking the theoretical parameters governing LLM scaling laws.

---

## 8. Conclusion

The paper presents "the first systematic study demonstrating that fine-grained per-token generation timings from production large language models can leak sensitive information." The authors frame this as "a significant threat" given the competitive value of such information, and note that defending against these timing side channels "without degrading latency or user experience remains an open and difficult challenge."

---

## Key Methodological Contributions

1. **Hybrid analytical-empirical modeling**: Theoretical asymptotic analysis combined with linear regression on empirical measurements yields explainable and generalizable runtime predictors.

2. **Correction for cuBLAS non-linearities**: A two-phase ensemble (classifier + per-kernel regressors) handles dynamic kernel selection, enabling generalization to unseen dimensions.

3. **Modular component-wise modeling**: Separate predictors for each subcomponent enable verification and targeted corrections.

4. **Two-stage ranking for KV-cache**: Leverages the more informative prefill signal to narrow the search before refining with both prefill and decoding.

---

## Notable Results Summary

| Attack | Target | Key Finding |
|--------|--------|-------------|
| Speculative decoding detection | Google Gemini Flash 2.5 | Speculative decoding confirmed; draft context ~128K |
| Speculative decoding detection | Google Gemini Flash 1.5 | Speculative decoding confirmed; draft context ~32K |
| Architecture leakage (L) | Llama 3.2 3B | Top-5 accuracy 86.15% (Eager) |
| Architecture leakage (H) | Llama 3.2 3B | Top-5 accuracy 71.54% (Eager) |
| Architecture leakage (H,L) | Llama 3.2 3B | Top-5 accuracy 65.38% (Eager) |
| Remote API (H known) | Llama 3.1 8B (W&B) | Correct H ranked #1 |
| Remote API (H,L) | Llama 3.1 8B (W&B) | Near-match ranked #8/2068 |
