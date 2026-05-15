---
url: https://thegeneralpartnership.substack.com/p/a-practical-guide-to-brownfield-ai
title: "A Practical Guide to Brownfield AI Development"
subtitle: "Context engineering for legacy codebases"
author: Daniel Pupius
date_fetched: 2026-05-15
date_published: 2026-02-04
publication: The General Partnership (Substack)
---

# A Practical Guide to Brownfield AI Development

Daniel Pupius challenges the conventional wisdom that AI coding agents are only suited for greenfield projects. He recounts an engineering lead whose team tried using Claude to refactor an 8-year-old Django monolith: the agent produced "clean, confident patches that quietly broke integrations with a couple of external services," leading to rollbacks and abandonment of the experiment.

The diagnosis: "real systems carry a lot of invisible context." The issue isn't solely model limitations — legacy systems lack the structural guardrails agents need. Greenfield codebases have clear boundaries and minimal hidden dependencies; brownfield ones have accumulated quirks and invisible contracts. This creates a bind: "agent autonomy requires structure, but legacy systems don't have it."

## Core Arguments

### Tests as System Boundaries
Linting, type-checking, and E2E tests create a "system boundary" enabling safe agent work. Pupius admits he historically avoided integration tests, but AI changes the tradeoffs. In one migration, he wrote 120+ Playwright tests against a vanilla JS app by having Claude inspect HTML/JavaScript to identify structural markers, then using Playwright to verify side effects of user interactions.

"AI agents excel at local transformations but lack global context." Tests provide negative feedback preventing drift.

### Documentation as Context
Tests tell agents *when* something breaks; documentation tells them *why* it was built that way. Pupius highlights "invisible contracts" — assumptions about data flow, integration quirks, and business rules encoded in forgotten code. He recommends agent-focused documentation: architecture overviews, integration maps. Standard CLAUDE.md / AGENTS.md files are helpful but insufficient for brownfield work.

He introduces a `/learn` command that lets agents, when they struggle, analyze failures and propose doc updates, creating compounding institutional memory.

### Incrementalism as Risk Management
Break migrations into "discrete, reversible phases": tooling introduction → structural extraction → framework migration → design system integration. "Agents perform best with bounded problems and clear success criteria." AI speed doesn't replace refactoring discipline — Pupius cites Martin Fowler and Kent Beck: "the fundamentals remain annoyingly fundamental."

### Compromise as Strategy
Not all technical debt is equally harmful. Pupius describes a hierarchy: security issues and data integrity are non-negotiable; existing functionality must not be broken; ugly patterns and tech debt are acceptable temporarily between steps. AI models "default to 'best practices' when you need 'what actually works.'" The skill is directing AI toward pragmatic, surgical changes.

### Structure as Enabler
The counterintuitive insight: "the faster you can fix things, the bolder you can be about breaking them." Structure that appears as overhead actually enables both speed and confidence. A React migration that "might have taken a week by hand was completed in a day."

## Caveats
1. Core architecture must still be conceptually valid — the data model must roughly fit the business.
2. Structure enables AI assistance but doesn't automate it. Agent autonomy ranged from 60%-95% depending on how well intent was communicated and how much context the agent could access autonomously.

Pupius characterizes the work as "more like managing a fast-moving team than running a script."

## Discussion Highlights
- Alireza Rahmani Khalili: "engineering trade-offs...treated seriously instead of being buried under AI hype aesthetics"
- Rainbow Roxy: "many legacy systems lack the structure agents need to operate safely" — human ingenuity needed for messy systems "a while longer"
