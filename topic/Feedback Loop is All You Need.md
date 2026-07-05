# Feedback Loop is All You Need

The argument that AI coding agents cannot be safely deployed through instructions alone. Deterministic feedback mechanisms — linters, CI/CD pipelines, observability — are what actually keep agent output honest. "Your CLAUDE.md is a suggestion. Your linter isn't."

---

## Key Quotes

> "Your CLAUDE.md is a suggestion. Your linter isn't."

The article's thesis compressed to seven words. CLAUDE.md explains _why_ and helps the agent get it right on the first try; a lint rule makes sure it _can't_ get it wrong. Instructions are "firmly in pirate code territory."

> "Encode rules once, let agents iterate against them, observe what fails, tighten the constraints. Less 'remember this next time,' more 'this literally cannot happen.'"

The shift from manual review to systematic enforcement. Every CI failure becomes a candidate for a new rule. The system tightens itself.

> "LLMs are probabilistic. They'll get it right most of the time, and on a real codebase 'most of the time' will eventually ruin your Friday night."

The probabilistic nature of LLMs isn't a bug you fix with better prompting — it's a property you defend against with deterministic tooling.

> "Linters don't sleep, and CI doesn't get tired."

Human review has limits. Automated enforcement doesn't.

> "The goal is to claim the leverage from the use of agents but without any compromise on the quality of the software." — Andrej Karpathy, cited in the article

The constraint that makes the whole thing a real engineering problem rather than a vibes exercise.

## Key Themes

#concept #guardrails #ci-cd #linting #agent-quality #agentic-coding #pattern

Ernie works at Archive.com, which went through three design systems in four years (Shopify Polaris → Ant Design → shadcn/ui + Tailwind). Agents regularly generate code that "imports from all three design systems in one file and somehow passes every check."

**The real enemy is invisible tech debt.** Code that compiles, passes tests, looks fine in review, and quietly violates architectural assumptions — importing from deprecated frameworks, using magic numbers instead of design tokens, `console.log` instead of structured logging.

**The highest-impact intervention** is brutally strict complexity constraints. ESLint rules for `max-lines-per-function` (40), `complexity` (10), `max-depth` (3), plus SonarJS's `cognitive-complexity`. When enforced, agents naturally decompose code into smaller testable units. Better architecture emerges from constraints, not instructions.

**The stack that works**: ESLint + SonarJS + strict TypeScript + opinionated React constraints + Prettier for code; Playwright screenshot tests + Chromatic for visual regressions; property-based testing for behavioral coverage; Sentry + Datadog for runtime monitoring that feeds back into agent task queues.

**The self-tightening loop**: Agent → Rules → CI → Observability → Tasks → Agent. Each CI failure becomes a new rule. The system gets smarter over time without human intervention.

### The Maturity Model

| Level | Description | Tell |
|-------|-------------|------|
| 0 — Vibes | No custom linting, manual review only | Eyes between agent and production |
| 1 — Guardrails | Standard linters + CI, no custom rules | Passes lint but drifts architecturally |
| 2 — Architecture as Code | Custom lint rules encode conventions | CLAUDE.md rules migrate into linter |
| 3 — The Organism | Self-tightening loop | Schedule agents overnight, review diffs in morning |

Most teams are at Level 0 or 1. The article argues Level 2 is the minimum to ship agent code safely.

## Critical Analysis

This is the single most actionable article in the wiki for anyone doing agent-driven development. Every other piece about agent quality is downstream of this insight: **the agent was waiting for better sensors, not a better model.**

The data is damning but not surprising. CodeRabbit: 1.7x more bugs, 2.74x more security vulnerabilities in AI code. DryRun Security: 87% of AI PRs contained vulnerabilities. These numbers would be career-ending for a human team, but they're treated as the cost of doing business with AI. Ernie's argument is that they don't have to be.

The maturity model is the most useful part. It gives you a roadmap, and it's honest about where most teams actually are (Level 0 or 1) vs. where they need to be (Level 2 minimum). "Schedule agents overnight, review diffs in the morning" is a concrete aspiration, not a vague promise.

The title's reference to "Attention Is All You Need" is well-earned. Just as attention mechanisms were the key insight for transformers, feedback loops are the key insight for deploying agents safely. Not better models, not better prompts: better sensors.

The economics are unanswerable. $200/month in tooling that catches one production bug per quarter has already paid for itself. A custom lint rule takes an afternoon to write and prevents an entire bug class forever. Senior engineers cost $150-200/hour. If you're not doing this, you're paying for it in ways you won't see until Friday night.

The article is strongest on linting and CI, weaker on observability and the "Tasks" link in the loop — how exactly do Sentry issues become agent tasks? The companion repo (vigiles) may fill that gap, but the article itself gestures at it.

The Karpathy framing — "leverage without quality compromise" — is the right north star. Most agent discourse oscillates between "go fast and break things" and "agents aren't ready." Ernie's position is that agents are ready, but _you_ aren't — and the fix is infrastructure, not a better model.

See also [[Harness Engineering]] for the theoretical framework behind this (feedforward vs. feedback, computational vs. inferential), [[Guardrails and Feedback Loops]] for the synthesis page, [[claude-ctrl]] for enforcement via hooks and SQLite, [[Pre-Commit Lint Checks]] for lint config as production infrastructure, [[Learn from PRs Skill]] for closing the feedback loop automatically, [[Compound Engineering]] for when to add a system instead of manual review, [[Write Only Code]] for the slop radius framing, [[Benchmark Exploitation]] for why evals alone don't catch this, [[Demystifying Evals for AI Agents]] for rigorous eval design, [[Writing a Good CLAUDE.md]] for what CLAUDE.md can and can't do, and [[Designing Agentic Loops]] for Simon Willison on choosing the right guardrails.

---
*Sources: [[summary/feedback-loop-is-all-you-need]]*
*Last updated: 2026-05-15*
