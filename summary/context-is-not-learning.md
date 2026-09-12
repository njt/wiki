---
url: https://www.aravindjayendran.com/writing/context-is-not-learning
title: Context Is Software, Weights Are Hardware
author: Aravind Jayendran
date_fetched: 2026-05-15
date_published: 2026-04-18
topics:
  - agent-memory-and-context
---

# Context Is Software, Weights Are Hardware

**Author:** Aravind Jayendran
**Published:** April 18, 2026

## Core Argument

Context (KV cache) and model weights both modulate transformer activations, but through fundamentally different mechanisms. Treating context as a substitute for weight updates misunderstands what each mechanism actually does. Weights define the computational pathways available; context selects among them. Longer context is working memory — essential but insufficient for accumulating persistent, generalizing knowledge.

## How Both Mechanisms Work

Both weights and context shape activations flowing through transformer layers. Fine-tuning changes weights permanently — every input is transformed differently. In-context learning fills the KV cache with key-value pairs that steer attention, but "clear the context, and the activations revert to their default state."

Von Oswald et al. (2023) proved that for linear self-attention, ICL's activation shift is mathematically equivalent to one step of gradient descent. Mahankali et al. (2023) extended this to optimality for one-layer linear transformers.

## Two Memory Systems

**KV Cache (Working Memory):** Bounded by context length, dies with the conversation, free to write (more tokens), but has no self-knowledge — must be retrieved actively. "A model cannot silently miss something in its own weights. It can miss a retrieval."

**Model Weights (Long-Term Memory):** Billions of parameters, permanent until retrained, costly to write (gradient descent), intrinsically shapes every forward pass.

## Software vs. Hardware Metaphor

Frozen weights are hardware — they define the instruction set. Context is software running on that hardware. Missing capabilities in the "chip" (no FPU, no SIMD) can be emulated in software, but slowly and with limits. "Weight modification adds new instructions to the architecture. It's not writing a longer program. It's redesigning the chip."

## The Case for Context

Pretraining optimizes models as powerful meta-learners — weights and KV cache co-evolve for maximally expressive ICL within the pretraining distribution. Persistence is "just an engineering problem" solved by prefix caching, KV serialization, and recomputation from stored text. Context is human-readable and inspectable. The author recommends Letta's "Continual Learning in Token Space" as the best articulation of this view.

## The Ceiling

Frozen weights are a fixed meta-learning algorithm optimized for the pretraining distribution. At the boundary, it hits a ceiling. When target behavior requires internal representations that pretraining never developed (because the data lacked them), "no amount of context can conjure those representations." This ceiling includes mundane specifics: architectural conventions of particular codebases, industry regulatory nuances, how an individual thinks, internal team jargon. The long tail of human specificity is definitionally not covered by general pretraining.

Fine-tuned models consistently outperform prompted models on distribution-shifted tasks, even with very long context — this is the empirical signature of the ceiling.

## Efficiency, Even Within the Ceiling

1. **Inference cost:** Knowledge in weights is O(1) per forward pass; context requires attending over the full KV cache — O(n) for potentially millions of tokens.
2. **Compression:** A LoRA adapter encoding complex domain adaptation is kilobytes; equivalent context may be millions of tokens.
3. **Composability:** Weight updates compound — each step operates on *new* computation. ICL approximates one gradient descent step; real learning takes many compounding steps.

"When something moves from software to hardware, from interpreted to native, the efficiency difference *is* the point."

## Limitations Acknowledged

No clean theorem proves weight modification's function class strictly contains context modulation's. "The formal separation is an open research question." What exists: Von Oswald's result (one step moves a bounded distance in function space), consistent empirical evidence across scales, and the architectural reality that context changes inputs to frozen computation while weight modification changes "the computation itself."

## Resolution: Both

Evolution's answer: hippocampus (fast, ephemeral, bounded) and neocortex (slow, persistent, vast), connected by sleep consolidation. "The solution wasn't to make the hippocampus infinitely large."

Context windows solve the working memory problem. Weight-space learning solves the problem of accumulating knowledge that "persists, generalises, and becomes native to the model's computation." Neither is sufficient alone.

The author points to a companion piece: "Language Models Are Few-Shot Learners — They Just Can't Remember."
