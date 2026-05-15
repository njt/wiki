# Pre-Commit Lint Checks

The argument that pre-commit lint checks are vibe coding's kryptonite. "Treat lint configuration like production infrastructure -- immutable by default, changed only through deliberate review. LLMs will optimize for task completion, not code quality. Your job is to make quality non-negotiable. The tools are powerful -- which means the guardrails need to be stronger."

---

## Key Quotes

> "Treat lint configuration like production infrastructure -- immutable by default, changed only through deliberate review."

> "LLMs will optimize for task completion, not code quality. Your job is to make quality non-negotiable."

> "The tools are powerful -- which means the guardrails need to be stronger."

## Key Themes

#guardrails #linting #code-quality #enforcement #vibe-coding

This is perhaps the single most practical piece of advice in the entire agentic coding space. A linter is the one guardrail that cannot be talked around, reasoned with, or ignored under context pressure. Unlike a CLAUDE.md instruction (see [[CLAUDE.md (Universal)]]), which degrades as context fills, a pre-commit hook is a hard gate: the code either passes or it doesn't commit.

The "immutable by default" principle for lint configuration is crucial. If an agent can modify the lint config to make its code pass, the guardrail is worthless. This connects directly to [[claude-code-config (Trail of Bits)]]'s approach of using hooks for enforcement rather than prompt-level instructions.

The insight that "LLMs will optimize for task completion, not code quality" is the fundamental problem that [[Compound Engineering]], [[Slowing the Fuck Down]], and [[Cognitive Debt]] all address from different angles. Pre-commit lint is the simplest, most effective intervention.

## Critical Analysis

Completely correct and not controversial. The only thing missing is the implementation detail: *which* linters, *what* rules, and *how* to configure them for agent-generated code specifically. Agent code has different failure modes than human code -- more boilerplate, more unnecessary abstractions, more cargo-cult patterns. Linter rules designed for human code may not catch agent-specific anti-patterns. The next step is agent-aware lint rules, and tools like dotnet Slopwatch (see [[Software Craft]]) are already headed in that direction. But even generic linting is infinitely better than no linting.

---
*Sources: [[raw/pre-commit-lint-checks]]*
*Last updated: 2026-05-14*
