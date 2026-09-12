# mira-OSS

A Python-based persistent AI agent framework built around the idea of "one conversation thread, forever." MIRA solves the stateless LLM problem through automatic memory decay, first-person narrative memory (the agent remembers "I debugged the IndexError" rather than "The assistant discussed debugging"), and a text-based LoRA system that evolves behavioral directives from accumulated feedback. It's an opinionated bet on continuity as the core design principle.

---

## Key Quotes

> "A comprehensive best-effort approximation of a continuous digital entity."

> "Has the potential to someday be something more than the sum of its parts."

## Key Themes

#personal-agents #memory #continuity #self-improving #open-source

The first-person narrative memory is MIRA's most distinctive idea. Most agent memory systems store third-person summaries ("The user asked about X, the assistant responded with Y"), which the model treats as external logs. First-person framing ("I debugged the IndexError in the config parser") encourages the model to treat memories as *experiences*, which changes how it integrates them into responses. This is a subtle but important design choice that connects to the memory retrieval problem discussed in [[Context Rot]] and [[Memory Mechanism]].

The text-based LoRA is fascinating: every seven active-use days, accumulated feedback (prediction errors, corrections, positive signals) gets synthesized into evolved behavioral directives. This is runtime personality evolution without model fine-tuning. It's a more systematic version of what [[Hermes]] calls "self-improving skills."

The dynamic tool management -- unused tools expire from context after five turns -- is pragmatic context engineering. Every tool description in the context window costs tokens. If the agent hasn't used a tool recently, remove it until needed.

## Critical Analysis

The "one conversation, forever" design is bold and has real trade-offs. It means context compression is critical -- you can't keep everything, so the memory decay and segment collapse systems are doing heavy lifting. The 120-minute SegmentCollapseEvent trigger is an interesting heuristic but feels arbitrary.

The Claude-first design (especially Opus 4.5) means this framework deeply exploits Claude's specific capabilities. That's a strength for Claude users and a weakness for everyone else. Compare with [[Hermes]], which supports any LLM but can't optimize for any specific one.

The AGPL-3.0 license and the creator's commitment to open-source parity with the hosted version is admirable. The "labor of love" framing and resistance to "shipping slop" comes through in the architecture -- this is clearly built by someone who uses it daily and cares about the details.

The PostgreSQL + pgvector + Valkey + Vault stack is heavy for a personal agent. The Docker deployment simplifies this, but it's a lot of infrastructure for what is fundamentally a personal assistant.

---
*Sources: [[summary/mira-oss]]*
*Last updated: 2026-05-14*
