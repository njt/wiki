---
title: "The Dark Factory is a DOT File"
url: https://2389.ai/posts/the-dark-factory-is-a-dot-file/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - specifications-as-the-product
  - agent-architecture
---

AI-powered software development is converging on a three-layer architecture: LLM client, agent loop, pipeline engine. The pipeline specification files (DOT format) are the truly valuable, reusable artifacts -- not the runner implementations.

The Attractor Pattern: multiple independent teams (StrongDM, Dan Shapiro, 2389 Research) built separate implementations and converged on identical architectural layers without coordination.

Software as Disposable: generated code is cheap and disposable. Specifications are the expensive part. When code is fundamentally wrong, rebuild from specs rather than debug.

Two pipeline styles: (1) Tool-based pipelines using shell commands for deterministic operations (seconds, no token expense), (2) LLM-based pipelines where every node calls a language model (expensive, slow, nondeterministic). Mix both strategically.

Products: Mammoth (DOT-based Go pipeline runner), Smasher (lean Rust with web dashboard), Tracker (Go with terminal UI), dotpowers (full SDLC in a single 53-node DOT file).

"The dark factory. Lights off. Nobody reviews the code. Nobody even looks at it."

"Software is cheap now. Specs are the expensive part."

"The factory code is dorodango -- polish it, throw it away, rebuild from spec."

"The question isn't how to build the factory anymore. It's what to build with it."