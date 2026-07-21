---
url: https://thegustafson.com/series
title: Holding the LLM Stack in Your Head
author: Nick Gustafson
date_fetched: 2026-07-21
date_published: unknown
---

# Holding the LLM Stack in Your Head — Nick Gustafson

A dependency-ordered walk through the modern LLM stack, "from the linear algebra under a single attention head" through training, inference, and agent protocols expected in 2026. Spans ten arcs and "eighty-odd posts." The stated goal: "it's intuition that survives contact with real systems."

The author notes this was written "as a learning exercise, in collaboration with AI (Claude Opus 4.8)." He describes it as "me thinking out loud with a model to understand the stack end to end" — not an authoritative reference — and advises readers to "take it with a grain of salt" and double-check anything they rely on.

## The Twelve Arcs

1. **01** — Mathematical & Computational Prerequisites (8 posts)
2. **02** — Language Modeling Before Transformers (7 posts)
3. **03** — Tokenization & the Input Pipeline (7 posts)
4. **04** — Transformers from First Principles (9 posts)
5. **05** — Decoding & the Real Inference Algorithm (9 posts)
6. **06** — Inference Engines & Serving Systems (9 posts)
7. **07** — Training & Post-Training (10 posts)
8. **08** — Evaluation & Scientific Discipline (7 posts)
9. **09** — Retrieval, Memory & Context Engineering (9 posts)
10. **10** — Tools, Protocols & Agent Loops (9 posts)

## Arc 01 — Detailed Posts

- "Vectors, Matrices, and the Spaces They Live In" — Vectors as lists of activations, matrix multiplication as a linear map, and why every neural network operation bottoms out in matmuls.
- "Norms, Dot Products, and Similarity" — Cosine similarity, L2 distance, projections, and why they appear in attention scores and embedding retrieval.
- "Distributions, Softmax, and the Chain Rule of Words" — Softmax, categorical distributions, Bayes' rule, and the chain rule of probability.
- "Cross-Entropy, KL Divergence, and What Loss Functions Measure" — Why cross-entropy is the standard LM loss, what it measures about distributions, and its connection to perplexity.
- "Gradients and How Machines Learn" — What a gradient is, why it points uphill, how backpropagation uses the chain rule, and what SGD does.
- "Optimizers: Momentum, Adam, and Learning Rate Schedules" — Why vanilla SGD is too slow, how Adam adapts per-parameter, and how warmup and cosine decay shape training.
- "GPUs, Floating Point, and Why Precision Matters" — IEEE 754, fp32/fp16/bfloat16 differences, why mixed-precision works, and GPU parallelism basics.
- "A Short Prehistory of Statistical NLP" — The arc from rule-based systems through statistical MT and log-linear models to neural approaches.

## Curated Learning Paths

**Path 1: Understanding Attention** — The minimum path to really seeing how a transformer layer works: Vectors/Matrices → Self-Attention: Q, K, V from First Principles → Multi-Head Attention and Representation Subspaces.

**Path 2: Why Inference Is Slow** — From the one-new-row insight through KV cache and the systems built around it: Prefill vs. Decode → Why One New Token Means One New Row → The KV Cache from First Principles → PagedAttention: Virtual Memory for the KV Cache.

**Path 3: Building with RAG** — Just enough retrieval theory to make engineering decisions make sense: Embeddings from Scratch (Word2Vec to E5) → Chunking Strategies → Rerankers and Cross-Encoders → RAG Architectures End to End.

**Path 4: Building Agents** — The loop, the protocol, the transcript formats, and what the model actually sees: A Short History of Agents (ReAct to 2026) → Function Calling as Structured Generation → The Agent Loop: Model, Runtime, Tool, Resume → MCP: A Cross-System Standard for Tool Integration.

## Closing Note

"The series is a full first draft" — the author will be "grinding through polish, corrections, and the odd rewrite."

Quote: Asimov — "The most exciting phrase in science is not 'Eureka!' but 'That's funny...'"
