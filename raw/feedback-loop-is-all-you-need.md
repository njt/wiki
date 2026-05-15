---
title: "Feedback Loop is All You Need"
url: https://zernie.com/blog/feedback-loop-is-all-you-need/
date_fetched: 2026-05-14
section: "Random"
---

# The Feedback Loop Is All You Need

Ernie's argument that AI coding agents cannot be safely deployed at scale through instructions alone. Deterministic feedback mechanisms -- linters, CI/CD, observability -- are essential.

Context: Working at Archive.com across four years and three design systems (Polaris -> Ant Design -> shadcn/ui). Agents generate syntactically correct code that violates architectural conventions invisibly.

"Your CLAUDE.md is a suggestion. Your linter isn't."

Highest-impact change: strict complexity constraints. ESLint: max-lines-per-function: 40, complexity: 10, max-depth: 3. Forces agents to decompose into smaller testable units. Better architecture emerges naturally.

Examples: 10-line lint rule forbidding console.log (suggest logger.error); Playwright screenshot tests catching layout shifts invisible to unit tests; property-based testing generating hundreds of test cases.

Data: CodeRabbit found AI code had 1.7x more bugs and 2.74x more security vulnerabilities. DryRun Security: 87% of AI PRs contained vulnerabilities. Spotify's Honk agent merges 650+ PRs/month after three years of feedback infrastructure. Devin's merge rate doubled (34% -> 67%) via better codebase understanding, not model improvements.

The feedback loop: Agent -> Rules -> CI -> Observability -> Tasks -> Agent. Each CI failure becomes a new linting rule. System tightens itself.

Four maturity levels:
- Level 0 (Vibes): Manual review only
- Level 1 (Guardrails): Standard linters + CI
- Level 2 (Architecture as Code): Custom lint rules encode conventions
- Level 3 (The Organism): Self-tightening loop, agents run overnight

"LLMs are probabilistic. They'll get it right most of the time, and on a real codebase 'most of the time' will eventually ruin your Friday night."

$200/month tooling that catches one production bug per quarter has already paid for itself. Senior engineers cost $150-200/hour; a lint rule prevents an entire bug class forever.
