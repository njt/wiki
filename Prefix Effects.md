# Prefix Effects

Research demonstrating that early naming decisions in a codebase create a "gravity effect" that shapes all subsequent AI-generated code. A function name is a seed: `secure_create_user` makes an agent construct a world with passwords and bcrypt, even without being asked. Different prefixes produce measurably different architectures.

---

## Key Quotes

> "A function name is a seed. `secure_create_user` makes an agent construct a world with passwords and bcrypt."

> "Early patterns in the codebase solidify over time as agents work on them."

## Key Themes

#agentic-coding #guardrails #cognitive-debt

Three findings:

**Vocabulary crystallization** -- AI-generated codebases lock in naming patterns rapidly. TF-IDF similarity across commits showed one codebase shifting from 8% to 81% alignment in a single commit. Human-generated codebases showed irregular fluctuations.

**Prefix propagation** -- agents automatically extend prefixes to new functions in the same domain. `secure_create_user` begets `secure_upload_document` without instruction. But propagation weakens across domain boundaries.

**Structural divergence** -- different prefixes produce different code. `secure_` triggers password fields and bcrypt. `energetic_` produces 54% more decorators and asyncio. `safe_` generates 53% more comprehensions. The prefix doesn't just name; it shapes architecture.

This has immediate practical implications: the first code committed to a repo is disproportionately important when agents will write the rest. Choose naming conventions deliberately. This connects to [[Talking to Transformers]] (Taylor's "railroad the model" advice is about choosing prefixes deliberately) and [[Simplicity in the Age of AI-Assisted]] (LLMs reproduce existing patterns faithfully, so your starter patterns determine everything).

The concept of naming as "alignment surface" is particularly interesting for [[Spec-Driven Development]] -- if function names steer agent behavior, then a spec that includes naming conventions is doing double duty as a guardrail.

## Critical Analysis

Solid research with concrete numbers. The finding that different prefixes produce measurably different architectures is surprising and actionable. This should change how teams think about coding standards in agent-assisted codebases -- naming conventions aren't just style, they're control surfaces.

The limitation: the research uses simple, well-understood patterns (CRUD apps with FastAPI/SQLAlchemy). Would prefix effects be as strong in a complex, domain-specific codebase where the agent has less training data to draw on? Likely weaker, but the direction of the effect would hold. More research needed on whether deliberate prefix engineering can steer agents toward desired architectural patterns at scale.

---
*Sources: [[raw/prefix-effects]]*
*Last updated: 2026-05-14*