# Spec-Driven Development

Drew Breunig's argument that specs, tests, and code form a triangle -- not a pipeline. Implementing code generates decisions that update the spec, which requires new tests, which surface new behaviors. The three nodes must stay synchronized, and the tooling's job (and the human's job) is keeping them in sync. Introduces Plumb, a tool to enforce this.

---

## Key Quotes

> "The spec defines what tests need to be written, and what code needs to be written. Tests validate the code. But the act of implementing code generates new decisions. Those decisions inform the spec."

> "When you can't see over your code, you can't oversee your code." (Hamilton's Law)

> "Our current Software Crisis is our inability to manage complex codebases new models allow."

> "If we improve the code, we must improve the spec."

## Key Themes

#spec-driven #guardrails #agentic-coding #testing

The triangle model rejects the waterfall assumption that specs flow one-way into code. Instead, code implementation clarifies intent -- it's not just execution. Specs are living documents updated by what the code reveals. This is the same insight as [[Spec-First Development at Benchling]]: the spec is the integration point, not the code.

**Plumb** (the tool) enforces this through git hooks: intercepts commits, extracts decisions from code diffs and agent traces, generates decision logs with intent documentation, maps spec requirements to code and tests, and prevents commits until decisions are reviewed. Decision capture transforms code review from "does this code work?" to "what did this code decide?"

The historical framing is interesting: Hamilton's Apollo work and the 1960s Software Crisis as parallels to today's AI-enabled complexity explosion. Every era oscillates between unhindered velocity and managed process.

This connects to [[OpenSpec]] (the open-source implementation of similar ideas), [[When the Target Keeps Moving]] (discovery vs. delivery maps directly to spec-update vs. code-commit), and [[Write Only Code]] (if nobody reads the code, the spec becomes the only human-readable artifact).

## Critical Analysis

The triangle model is the right mental model. Linear spec-to-code pipelines have always been fiction; making the feedback loop explicit and tooled is overdue. Plumb's approach of intercepting commits and extracting decisions is clever -- it adds process without adding ceremony, because the decisions are extracted automatically.

The risk: decision extraction from diffs is hard. How does Plumb distinguish between trivial implementation choices and meaningful architectural decisions? If it flags everything, it becomes noise. If it only catches obvious things, it misses the subtle choices that matter most. The answer probably involves LLM classification, which brings its own reliability questions.

---
*Sources: [[raw/spec-driven-development]]*
*Last updated: 2026-05-14*