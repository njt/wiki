---
title: "Self-Distillation"
url: https://arxiv.org/abs/2604.01193
date_fetched: 2026-05-14
section: "LLMs"
topics:
  - ai-research-and-models
---

# Simple Self-Distillation for Code Generation

ArXiv paper showing a model can improve at code generation using only its own raw outputs, without a verifier, teacher model, or reinforcement learning.

## Method

Simple Self-Distillation (SSD): sample solutions from the model with certain temperature and truncation configurations, then fine-tune on those samples with standard supervised fine-tuning.

## Results

SSD improves Qwen3-30B-Instruct from 42.4% to 55.3% pass@1 on LiveCodeBench v6, with gains concentrating on harder problems. Generalizes across Qwen and Llama models at 4B, 8B, and 30B scale, including both instruct and thinking variants.

## Key Insight

A "precision-exploration conflict in LLM decoding" can be resolved through self-distillation, which "reshapes token distributions in a context-dependent way, suppressing distractor tails where precision matters while preserving useful diversity where exploration matters."

## Rationale (from annotation)

Key token moments are locks (only one right answer) and forks (lots of right answers). Generate one solution at high temp for each of 10k problems to get creative answers without distracting possibilities; then fine-tune on those outputs to lock in the distractor cleanup; then generate answers to normal prompts at lower temperatures. This process cleans up locks and expands the model's creativity at forks.

## Methodology

Two stages:
1. Sampling phase: Generate solutions using specific temperature and truncation settings
2. Fine-tuning phase: Standard supervised fine-tuning on the generated samples

No external verifiers, teacher models, or reinforcement learning required.

## Conclusion

Self-distillation provides "a complementary post-training direction for improving LLM code generation" and demonstrates models can self-improve through their own outputs alone.
