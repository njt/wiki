# Feedback Loop is All You Need

The argument that AI coding agents cannot be safely deployed through instructions alone. Deterministic feedback mechanisms -- linters, CI/CD pipelines, observability -- are what actually keep agent output honest. "Your CLAUDE.md is a suggestion. Your linter isn't."

---

## Key Quotes

> "Your CLAUDE.md is a suggestion. Your linter isn't."

> "Encode rules once, let agents iterate against them, observe what fails, tighten the constraints. Less 'remember this next time,' more 'this literally cannot happen.'"

> "LLMs are probabilistic. They'll get it right most of the time, and on a real codebase 'most of the time' will eventually ruin your Friday night."

## Key Themes

#concept #guardrails #ci-cd #linting #agent-quality #agentic-coding

Ernie's experience at Archive.com across four years and three design systems (Polaris -> Ant Design -> shadcn/ui) is the evidence base. Agents generate syntactically correct code that silently violates architectural conventions -- importing from deprecated frameworks, using magic numbers instead of design tokens, adding console.log instead of structured logging.

The highest-impact intervention: strict complexity constraints. ESLint rules for max-lines-per-function (40), complexity (10), max-depth (3). When these are enforced, agents decompose code into smaller testable units naturally. Better architecture emerges from constraints, not instructions.

The holy grail is a self-tightening loop: Agent -> Rules -> CI -> Observability -> Tasks -> Agent. Each CI failure becomes a new rule. The system gets smarter over time without human intervention.

The data is damning: CodeRabbit found AI code had 1.7x more bugs and 2.74x more security vulnerabilities than human code. DryRun Security: 87% of AI PRs contained vulnerabilities. But the success stories prove the fix works: Spotify's Honk agent merges 650+ PRs/month after three years of feedback infrastructure investment.

## Critical Analysis

This is the most important article in the batch for anyone doing agent-driven development. The maturity model (Vibes -> Guardrails -> Architecture as Code -> The Organism) gives you a roadmap. Most teams are at Level 0 or 1; the article argues you need Level 2 minimum to ship agent code safely.

The economics argument is strong: $200/month in tooling that catches one production bug per quarter pays for itself many times over. A custom lint rule takes an afternoon to write and prevents an entire bug class forever. Senior engineers cost $150-200/hour.

The title's reference to "Attention Is All You Need" is apt -- just as attention mechanisms were the key insight for transformers, feedback loops are the key insight for deploying agents safely. Not better models, not better prompts: better sensors.

See also [[Optimise Anything]] for automated optimization that could feed into this loop, [[agent-pr-replay]] for empirical measurement of agent gaps, and [[session-analysis]] for the observability piece.

---
*Sources: [[raw/feedback-loop-is-all-you-need]]*
*Last updated: 2026-05-14*
