---
url: https://typesafe.ai/blog/introducing-system-one-models-and-jev
title: "Introducing System One Models & Jev"
author: Diogo Almeida
date_fetched: 2026-09-19
date_published: unknown
topics:
  - ai-research-and-models
  - mcp-and-tool-protocols
---

# Introducing System One Models & Jev

Diogo Almeida (ex-OpenAI, research behind ChatGPT's instruction-following work) announces TypeSafe AI's first product after two years in stealth: a new class of frontier models called **System One Models**, built to make fast, structured decisions that software can call directly. The first public model, **Jev** (named for William Stanley Jevons), matches LLM-level intelligence on "System One" tasks while claiming to be roughly two orders of magnitude faster and cheaper — the home-page figures are 193.6× faster and 444.6× cheaper.

The technical bet is an inversion of the LLM: Jev does not generate strings autoregressively at all. It outputs all probabilities in parallel — "unstructured state in, typed probabilistic decisions out." The stack includes a new model architecture, a parallel sampler, and a training method called Reinforcement Learning for Calibrated Decisions (RLCD), which optimizes for epistemically honest probabilities rather than human-preferred prose or verifiable rewards. Because outputs are never generated as strings, the company claims schema conformance is guaranteed by construction — Jev "can't hallucinate" in the type-error sense — and every output carries calibrated confidence.

The evidence offered is candid about its limits. A side-by-side demo against GPT-5.6 Terra shows parallel vs. autoregressive sampling on a deliberately simplified query. New "workflow evals" score models inside a fixed code workflow against the average of the largest external models (GPT-6 Astra and Fable 5.1), with the authors admitting the reference choice biases toward OpenAI/Anthropic and that the eval workflows were built by their own capabilities team. The "no type errors" claim is definitional ("Our number is not empirical. Schema matching is guaranteed"), and pricing transparency comes with the admission that subsidy can't be ruled out. Fun demos include a Doom bot issuing 10 queries/second (~$7/hour) and a wikiracing agent exploiting high-cardinality choices.

The framing is Kahneman's Thinking, Fast and Slow: LLMs are System 2 (slow, deliberate, generative); System One Models are the fast intuitive layer, and the company argues the System-1-is-error-prone association can be reversed through calibrated training. The closing thesis: "AI needs an interface software could depend on" — and every order-of-magnitude drop in the cost of intelligence unlocks new use cases, Jevons-paradox style.
