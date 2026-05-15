# OpenSpec

An open-source, spec-driven framework that serves as a universal planning layer for AI coding agents. Specs live in your code alongside implementations, persist across sessions, and produce spec deltas that capture how requirements evolve -- making it possible for reviewers to understand intent without digging through implementation.

---

## Key Quotes

> "Specs only work if you actually read them, think through them, and engage with them."

## Key Themes

#spec-driven #agentic-coding #devtools

Three core capabilities:

**Intent-based review** -- each change produces a spec delta showing how requirements evolved. Reviewers see the change in intent, not just the change in code. This directly addresses the coordination problem in [[Zero Alignment]] -- if the spec delta is clear, review doesn't require reading all the code.

**Persistent context** -- specs organize by capability within the repo structure. New developers understand system architecture through browsable documentation rather than institutional knowledge. This is a practical answer to [[Nobody Knows How Large Software Projects Work]] -- make the system knowable through maintained specs.

**Rapid proposal generation** -- describe desired changes via agent commands, get back proposal documents, design decisions, implementation tasks, and requirement deltas before coding begins. Ten minutes of thinking before coding.

Integrates with 30+ coding platforms (Claude Code, Cursor, GitHub Copilot, Windsurf, Amazon Q) via native slash commands. Specs are version-controlled using standard git workflows.

Philosophy: minimal effort, lightweight process. Create specs incrementally as needed, not upfront for the entire codebase. This is more pragmatic than [[Spec-Driven Development]]'s triangle model -- OpenSpec doesn't try to enforce synchronization, it just makes specs cheap enough to maintain.

## Critical Analysis

OpenSpec occupies a sweet spot: more structured than ad hoc plan files, less heavy than formal requirements management. The "spec deltas" concept is particularly strong -- it's the diff for intent, not just code, which is exactly what code review needs in an agent-accelerated world.

The risk: adoption depends on developers actually reading and updating specs. The tool makes this cheaper but can't make it happen. "Specs only work if you actually read them" is both honest and a warning sign -- the history of software documentation is littered with tools that made documentation easy but couldn't make anyone care.

---
*Sources: [[raw/openspec]]*
*Last updated: 2026-05-14*