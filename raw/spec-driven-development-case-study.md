---
url: https://felipefontoura.com/articles/spec-driven-development-case-study/
title: "Spec-Driven Development Case Study: 13 Apps in 70 Days, Solo, with AI"
author: Felipe Fontoura
date_fetched: 2026-07-18
date_published: 2026-06-11
---

# Spec-Driven Development Case Study: 13 Apps in 70 Days, Solo, with AI

**Author:** Felipe Fontoura
**Publication Date:** June 11, 2026
**Tags:** Spec-Driven Development, AI Agents, Case Study, Claude Code, Fintech

---

## Overview

The article describes building a production-grade crypto payment platform — a PIX payment gateway for Brazil, an OTC exchange engine, and on-chain settlement on Bitcoin's Liquid Network — entirely solo over 70 days using AI agents guided by written specifications (Spec-Driven Development or SDD).

### Project Stats

- **13 apps** in a Turborepo monorepo
- **3 APIs, 3 databases** (auth, payments, exchange)
- **8 shared packages** (auth, ui, i18n, logger)
- **~1,650 automated tests** (Vitest + Playwright)
- **138K lines of TypeScript**, 39 migrations
- **70 days**, solo, handling real money

---

## The Bet

The brief was a complete, production-grade crypto payment platform — not a prototype. The author argues that the main objections to SDD come from toy projects where it seems like overhead, but notes that "A 10-hour solo project with no compliance pressure, no real balances, and no external deadline does not need a 28-file spec corpus."

He contrasts "prompt-and-pray" with specification: "it builds something that looks complete and passes a shallow review, then fails when a real edge case arrives."

---

## The Model Maker Says Plan First

Fontoura cites Anthropic's own guidance for Claude Code — a four-step loop of explore, plan, implement, commit — noting their reasoning that "letting Claude jump straight to coding can produce code that solves the wrong problem."

He references a 2025 METR randomized trial where experienced developers with AI were "19 percent slower than without it, despite expecting to be 24 percent faster." His conclusion: "Capability amplifies direction. It does not supply it."

The mechanism was a standing specification corpus of 28 markdown specs across 12 domains, plus 8 custom slash commands, all living in the repository and loaded as agent context each session.

---

## Spec First, Prompt Never

Every capability began with a written specification covering requirements, design, and tasks. Agents executed against the spec. When output was wrong, the author fixed the spec, not the prompt.

The spec corpus included domains for auth, exchange pricing, database design, testing conventions, UI tokens, codebase conventions, and custom commands. The auth domain, for example, covered "five roles, nineteen scopes, eighteen granular permissions" plus machine-to-machine flows.

He explains that "A single developer does not hold a 13-app fintech in their head. The spec holds it. The developer holds the spec."

---

## The Delivery Loop

The four-step loop repeated per capability:

1. **Specify** — Requirements, design, and tasks written before any code. Human approval gates advancement.
2. **Generate** — The agent implements against the spec, writing tests alongside code.
3. **Verify** — Review against acceptance criteria via Prettier, ESLint zero warnings, strict tsc, and tests on a live database.
4. **Correct the spec** — Wrong output means a wrong spec. Fix the document, regenerate. Never patch code and leave the spec behind.

He calls the fourth step "where the discipline lives," warning that "Patch the code, leave the spec untouched, and the next time that module is regenerated or extended, the agent rebuilds from the spec and reintroduces the same error."

---

## What the Specs Caught

### The Pricing Engine

The OTC exchange pricing engine used a two-phase model with a peg rate and a VWAP window requiring at least five confirmed trades in 24 hours. The critical precision rule required "all money arithmetic in 8-decimal integers (satoshi precision)" with float anywhere being a test failure, not a lint warning.

The spec included a fallback chain across five market sources, asymmetric per-pair spreads, and an explicit out-of-scope list that stopped the agent from adding unrequested features twice.

### Payment Idempotency

The idempotency rule was frozen before any charge endpoint code existed: a `UNIQUE(merchant_id, idempotency_key)` constraint enforced at the database level. He notes "Without a spec, that constraint lives in your head. It falls out on the next regeneration."

---

## Comparison Table

**Without specs (prompting):** Float arithmetic and rounding drift, silent scope creep, edge cases surfacing in production, idempotency constraint held only in memory, no record of decisions.

**With specs:** 8-decimal integer math enforced by test failure, negative scope prevented unrequested features, edge cases caught at review, database-level unique constraint, every decision traceable in git.

The author states: "The agent's capability was identical in both columns. The spec is the only variable."

---

## What This Proves, and What It Does Not

Fontoura is explicit about limitations:

- **One system, one developer, one deadline** — no control group, cannot isolate method from operator
- He brings 25 years of experience including fintech, aerospace, and enterprise integration; "SDD scaled a mature mental model"
- Speed was measured but quality was not independently audited; "Green tests and tight CI have shipped bad systems before"
- An external deadline was a real motivator
- "Production is not a business" — an engineering method, not a market proof
- "One case with a favorable outcome is not the same as a methodology with broad empirical support"

His structural argument: "an AI agent has no memory between sessions, and the spec is the external memory you give it."

He concludes: "Without the specs, the agent is a fast bricklayer with no blueprint. With them, it is a senior engineer with perfect recall of every decision you made."

---

## Closing

The article ends with links to further SDD resources and a newsletter tagline: "Don't Code, Specify. A weekly dispatch from where AI agents meet real production."

Final line: "The code wrote itself. The specs did not. That is where the seventy days went, and it is why they were enough."
