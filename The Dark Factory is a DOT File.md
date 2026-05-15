# The Dark Factory is a DOT File

The argument that AI-powered software development is converging on a standard three-layer architecture (LLM client, agent loop, pipeline engine), and that the pipeline specification -- a DOT file describing the workflow -- is the valuable, reusable artifact. The factory code is disposable. The specs are the product.

---

## Key Quotes

> "The dark factory. Lights off. Nobody reviews the code. Nobody even looks at it."

> "Software is cheap now. Specs are the expensive part."

> "The factory code is dorodango -- polish it, throw it away, rebuild from spec."

> "The question isn't how to build the factory anymore. It's what to build with it."

## Key Themes

#orchestration #agentic-coding #spec-driven

The convergence evidence: StrongDM, Dan Shapiro ([[Trycycle]]), and 2389 Research built separate implementations in different languages and arrived at the same three-layer architecture independently. This suggests the design solves a fundamental problem.

Two pipeline styles:
- **Tool-based** -- shell commands, deterministic, cost-free, seconds to run
- **LLM-based** -- every node calls a language model, expensive, slow, nondeterministic

Mix them strategically. Not everything needs an LLM; many pipeline steps are just "run the tests" or "commit the code."

The DOT file itself is standard Graphviz syntax. Nothing proprietary. Each node describes whether it needs an LLM, a human gate, parallel branches, or verification commands. The pipeline is the reusable blueprint; the runner is the disposable implementation.

Products: Mammoth (Go), Smasher (Rust with web dashboard), Tracker (Go with TUI), dotpowers (53-node full SDLC in one DOT file).

This is the infrastructure layer for [[Trycycle]], [[Verbose Deployment]], and similar pipeline-based tools. The philosophy connects directly to [[Simplicity in the Age of AI-Assisted]] (code is disposable, specs are durable) and [[Spec-Driven Development]] (the spec is the integration point).

## Critical Analysis

The "specs are expensive, code is cheap" framing is the single most important idea in this batch. It inverts the traditional economics of software: we used to invest heavily in code and treat specs as overhead. If code generation is near-free, the spec becomes the asset.

The DOT format choice is clever -- human-readable, tool-supported, version-controllable, and familiar to anyone who's used Graphviz. The risk: DOT is a graph description language, not a workflow language. It lacks constructs for error handling, conditional branching, timeouts, and retry logic that real pipelines need. The current implementations presumably encode these in node attributes or conventions, which means the "standard" is really "DOT plus conventions" -- less portable than pure DOT suggests.

---
*Sources: [[raw/the-dark-factory-is-a-dot-file]]*
*Last updated: 2026-05-14*