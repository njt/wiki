---
title: "The Feedback Loop Is All You Need"
author: Ernie
url: https://zernie.com/blog/feedback-loop-is-all-you-need/
date_fetched: 2026-05-15
date_published: 2026-03-27
companion_repo: https://github.com/zernie/vigiles
discussion: https://news.ycombinator.com/item?id=47618610
topics:
  - guardrails-and-feedback-loops
---

# The Feedback Loop Is All You Need

Ernie's argument that AI coding agents cannot be safely deployed at scale through instructions alone. Deterministic feedback mechanisms — linters, CI/CD, observability — are what actually keep agent output honest. The title riffs on "Attention Is All You Need" (Vaswani et al., 2017).

Context: Ernie works at Archive.com, where four years of work produced three design systems (Shopify Polaris → Ant Design → shadcn/ui + Tailwind). Claude Code's new scheduled agents couldn't be applied to their real codebase because agents generate code that "imports from all three design systems in one file and somehow passes every check."

## Core Argument

The real threat isn't agents breaking things visibly — it's "invisible tech debt": code that compiles, passes every test, looks fine in review, and quietly violates architectural assumptions. Examples: agents reverting migration progress by pulling from old design systems, copying magic padding values (`p-[24px]` instead of `p-6`) from adjacent components.

**CLAUDE.md vs. Linters**: "CLAUDE.md explains the why and helps the agent get it right on the first try. A lint rule makes sure it can't get it wrong." CLAUDE.md is "firmly in pirate code territory" — a suggestion, not a guarantee. Vercel's agent-skills library is cited as an example of well-written instructions that are still not enforceable.

## Tool Stack

- ESLint, SonarJS (especially `cognitive-complexity`), strict TypeScript, opinionated React constraints, Prettier
- Highest-impact change: brutally strict complexity limits — cyclomatic complexity caps (`complexity`), nesting limits (`max-depth`), function length (`max-lines-per-function`), parameter limits (`max-params`), statement limits (`max-statements`)
- Playwright screenshot tests for visual regressions (z-index, layout shifts, unclickable buttons)
- Chromatic for Storybook-based visual testing
- Property-based testing — AI flipped the cost equation; defining properties agents generate is now feasible
- Runtime monitoring: Sentry and Datadog, wiring production issues into agent task queues

## Data Points

- CodeRabbit: AI-generated code has 1.7x more bugs, 2.74x more security vulnerabilities than human code
- DryRun Security: 87% of AI PRs had at least one vulnerability (tested Claude Code, Codex, Gemini)
- Study (arxiv.org/html/2510.09907v1): agents against 933 modules → 984 bug reports, 56% valid, ~$10 per real bug
- Spotify's Honk: 650+ agent-generated PRs to production per month, built on feedback infrastructure from 2022
- Devin's merge rate doubled (34% → 67%) after improving codebase understanding, not model capability

## The Feedback Loop

Agent → Rules → CI → Observability → Tasks → Agent. "Encode rules once, let agents iterate against them, observe what fails, tighten the constraints. Less 'remember this next time,' more 'this literally cannot happen.'"

## Maturity Model

| Level | Description | Tell |
|-------|-------------|------|
| 0 — Vibes | No custom linting, manual review | Eyes between agent and production |
| 1 — Guardrails | Standard linters + CI, no custom rules | Passes lint but drifts architecturally |
| 2 — Architecture as Code | Custom lint rules encoding conventions | CLAUDE.md rules migrate into linter |
| 3 — The Organism | Self-tightening loop | Schedule agents overnight, review diffs in morning |

Immediate actions: turn one recurring PR comment into a lint rule; add Playwright screenshot tests for critical pages; schedule an agent for safe tasks (dependency updates, branch cleanup).

## Key Quotes

> "Your CLAUDE.md is a suggestion. Your linter isn't."

> "Encode rules once, let agents iterate against them, observe what fails, tighten the constraints. Less 'remember this next time,' more 'this literally cannot happen.'"

> "LLMs are probabilistic. They'll get it right most of the time, and on a real codebase 'most of the time' will eventually ruin your Friday night."

> "The goal is to claim the leverage from the use of agents but without any compromise on the quality of the software." — Andrej Karpathy, cited in the article

> "Linters don't sleep, and CI doesn't get tired."

## Economic Argument

~$200/month in tooling that catches one production bug per quarter pays for itself many times over. A custom lint rule takes an afternoon to write and prevents an entire bug class forever. Senior engineers cost $150-200/hour. The real cost is time, not tokens or subscriptions.
