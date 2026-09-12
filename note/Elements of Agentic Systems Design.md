# Elements of Agentic Systems Design

William Chen's framework decomposing agentic systems into ten behavioral elements: Context, Memory, Agency, Reasoning, Coordination, Artifacts, Autonomy, Evaluation, Feedback, and Learning. Positioned as a design space map for builders of agent frameworks, SDKs, and platforms. The core thesis: "The model is a stateless text-to-text function. Everything else is architecture you build around it."

---

## Key Quotes

> "The model is a stateless text-to-text function. Everything else is architecture you build around it."

> Intelligent-seeming behavior traces to concrete code patterns, not emergent model properties.

## Key Themes

#agent-architecture #design-framework #taxonomy #evaluation #memory

The ten elements form a useful vocabulary for analyzing any agent system:

1. **Context** -- what the model sees per call (see [[Experience Design for Agents]] on drift)
2. **Memory** -- external storage for selective retrieval (see [[Chief of Staff]]'s three-layer memory)
3. **Agency** -- translation from text to effects (the tool layer)
4. **Reasoning** -- how calls compose (chaining, looping, branching)
5. **Coordination** -- multi-agent communication (see [[Cord]])
6. **Artifacts** -- shared persistent state (see [[LLM Wiki]]'s wiki pages)
7. **Autonomy** -- what triggers execution (see [[Ralph]]'s loop, [[MimiClaw]]'s heartbeat)
8. **Evaluation** -- measuring success (see [[LLM Evals]], [[Benchmark Exploitation]])
9. **Feedback** -- signals steering current behavior
10. **Learning** -- feedback that persists to change future behavior

The externalization pattern is a recurring insight: Memory externalizes Context, Artifacts externalize Coordination, Learning externalizes Feedback. Each element has a "persisted" version that survives across sessions.

## Critical Analysis

Strong: the taxonomy is genuinely useful for comparing agent architectures. You can point at any system in this batch and identify which elements it focuses on and which it ignores. The "behavior-to-code mapping" framing keeps it practical rather than academic. CC BY 4.0 licensing encourages adoption.

Weak: ten elements is arguably too many -- some feel like splits of a natural pair (Feedback/Learning, Context/Memory). The framework is descriptive rather than prescriptive: it tells you what elements exist but not how to choose between design options within each element. And the claim that "intelligent-seeming behavior traces to concrete code patterns, not emergent model properties" is partially wrong -- [[Emotion concepts and their function in a large language model]] shows that models develop internal representations that influence behavior in ways not predicted by the architecture.

Most useful as a shared vocabulary when discussing agent systems, less useful as a design guide when building them.

For a pattern catalog organized by constraint rather than taxonomy — the Alexander-esque "navigate from problem to solution" model applied to agent design — see [[Agentic Design (Pattern Catalog)]], KORTEXYA's free catalog of 280+ connected agent design patterns.

---
*Sources: [[summary/elements-of-agentic-system-design]]*
*Last updated: 2026-08-01*
