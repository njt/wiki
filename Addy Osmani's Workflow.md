# Addy Osmani's Workflow

Addy Osmani's comprehensive guide to integrating AI coding assistants into professional development. The core argument: classical software engineering practices become *more* critical, not obsolete, when you add AI. Start with a spec.md (requirements, architecture, data models, testing strategy), work in iterations (plan + implement + test + commit), and treat AI output like junior developer code -- always review, always test.

---

## Key Quotes

> "AI coding assistants are incredible force multipliers, but the human engineer remains the director of the show."

> "The LLM is an assistant, not an autonomously reliable coder...I am the senior dev."

> "At Anthropic, engineers adopted Claude Code so heavily that ~90% of the code for Claude Code is written by Claude Code itself."

> "Think of an LLM pair programmer as over-confident and prone to mistakes."

## Key Themes

#agentic-coding #spec-driven #testing #code-review #iteration #context-packing

The spec.md approach is the most transferable idea here. It's "waterfall in 15 minutes" -- use AI to rapidly iterate on requirements before touching code. The emphasis on examples as context (not just instructions) is practical and underappreciated. His "model musical chairs" suggestion -- switching between Claude, Gemini, and others when one stalls -- is pragmatic but adds cognitive overhead.

The iteration pattern (plan + implement + test + commit as one unit) connects to [[Planning With Files]]'s approach to maintaining state across sessions. Both are trying to impose structure on inherently chaotic AI-assisted development.

## Critical Analysis

This is the responsible, professional-grade take on AI coding. Almost everything here is sound advice. The tension is in the closing: "No matter how much AI I use, I remain the accountable engineer." That's true today, but [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] suggests we're heading somewhere that makes it less true. Osmani's workflow is solidly Level 2-3 in Shapiro's framework. The question is whether the discipline he advocates scales to Level 4+, or whether it becomes impossible to maintain human oversight at higher automation levels. See also [[The Next Two Years of Software Engineering]] (his companion piece on the macro trends) and [[14 More lessons from 14 years at Google]] for his organizational thinking.

---
*Sources: [[raw/addy-osmanis-workflow]]*
*Last updated: 2026-05-14*
