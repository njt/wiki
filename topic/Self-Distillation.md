# Self-Distillation

A paper showing that LLMs can improve at code generation using only their own outputs -- no verifier, no teacher model, no reinforcement learning. Simple Self-Distillation (SSD): sample solutions at high temperature, then fine-tune on those samples. Qwen3-30B goes from 42.4% to 55.3% pass@1 on LiveCodeBench v6, with gains concentrating on harder problems. Generalizes across Qwen and Llama at 4B-30B scale.

---

## Key Quotes

> "Reshapes token distributions in a context-dependent way, suppressing distractor tails where precision matters while preserving useful diversity where exploration matters."

## Key Themes

#training #self-improvement #distillation #code-generation #decoding

The intuition is elegant: every token generation moment is either a "lock" (one right answer) or a "fork" (many right answers). High-temperature sampling generates creative solutions that navigate forks well but also happen to clean up lock moments by filtering out distractors. Fine-tuning on these samples bakes in the cleanup, so normal-temperature generation gets both precision and creativity.

What makes this notable: the model improves itself with zero external signal. No human labels, no reward model, no stronger teacher -- just the model's own samples, filtered by the laws of probability. It's self-improvement through self-selection.

## Critical Analysis

Strong: the method is dead simple (sample + SFT) and the gains are substantial (+13 percentage points on LiveCodeBench). Generalizes across model families and scales. The lock/fork framework for understanding token distributions is conceptually clean.

Missing: this only works for code generation where correctness is somewhat verifiable through execution. Whether SSD generalizes to domains without clear correctness signals (creative writing, analysis, advice) is an open question. The paper also doesn't address whether repeated rounds of self-distillation converge or degrade -- can you SSD the SSD'd model?

The deeper question this raises: if models can self-improve through their own outputs, what's the long-term trajectory? Connects to alignment concerns about recursive self-improvement, though the gains here are modest and bounded.

**Contrast with verifier-based improvement.** Self-distillation needs no external signal — the model bootstraps from its own distribution. [[LLM-as-a-Verifier]] takes the opposite approach: an external verifier provides fine-grained continuous scores to select the best from multiple candidates. The two approaches are complementary: self-distillation improves the generator, verification improves selection among generator outputs. Combined, they'd address both halves of the improvement problem.

---
*Sources: [[summary/self-distillation]]*
*Last updated: 2026-07-18*
