---
title: "Theories of Deep Learning"
url: https://astledsa.substack.com/p/theories-of-deep-learning
date_fetched: 2026-07-18
section: "AI Research & Models"
---

# Theories of Deep Learning — astle dsa (Theoretical Limits)

A July 2026 Substack essay surveying three emerging mathematical theories that aim to explain why deep learning works, each targeting a different aspect: architecture (Categorical Deep Learning), optimization (Modular Duality), and generalization (output-space dynamical systems via eNTK). The author, writing with AI assistance, presents these as the strongest foundational frameworks narrowing the long-standing gap between deep learning's empirical success and its theoretical understanding.

## Three Theories

**Categorical Deep Learning** (Gavranović et al., arXiv:2402.15332) proposes an algebraic theory of all neural network architectures using the universal algebra of monads in a 2-category of parametric maps. It bridges the gap between constraint specification and implementation, representing sequential computation and recurrence through monad compositionality. Built on category theory concepts from functional programming.

**Modular Duality** (Bernstein & Newhouse, arXiv:2410.21265) addresses a mathematical error in optimization: gradients and weight updates live in different vector spaces (a norm's dual space vs. the original space), making naive subtraction incoherent. The solution is a Duality Map that transforms gradients into correct parameter-space updates. Each layer type gets its own norm, recursively composed into a Modular norm for the full architecture, yielding geometry-aware optimization.

**A Theory of Generalization** (Litman & Guo, arXiv:2605.01172v1) abandons parameter-space analysis entirely, treating the network as a dynamical system in output space. Using the empirical Neural Tangent Kernel, it partitions learning dynamics into a signal channel (high-eigenvalue, fast-learning) and a reservoir (low-eigenvalue, slow-learning). This single mechanism explains benign overfitting, double descent, implicit bias, and grokking.

## Other Efforts

The essay notes mechanistic interpretability (Anthropic's circuits, induction heads, J-space) as a related sub-field, and references several book-length mathematical treatments and an extended CDL thesis that claims to cover all aspects of deep learning.
