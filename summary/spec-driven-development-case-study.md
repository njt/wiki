---
url: https://felipefontoura.com/articles/spec-driven-development-case-study/
title: "Spec-Driven Development Case Study: 13 Apps in 70 Days, Solo, with AI"
author: Felipe Fontoura
date_fetched: 2026-07-18
date_published: 2026-06-11
topics:
  - specifications-as-the-product
  - agent-coding-workflow
---

Felipe Fontoura describes building a production-grade crypto payment platform — a PIX
gateway for Brazil, an OTC exchange engine, and on-chain settlement on Bitcoin's Liquid
Network — entirely solo over 70 days using AI agents guided by written specifications. The
result was 13 apps in a Turborepo monorepo (3 APIs, 3 databases, 8 shared packages),
~138K lines of TypeScript, and ~1,650 automated tests.

His core argument is Spec-Driven Development (SDD): every capability begins with a
written specification covering requirements, design, and tasks. Agents execute against the
spec. When output is wrong, the developer fixes the spec — not the prompt or the code.
Fontoura argues this is where the discipline lives; patching code while leaving the spec
untouched guarantees the same error reappears on the next regeneration.

The project used a standing corpus of 28 markdown specs across 12 domains (auth,
exchange pricing, database design, testing conventions, UI tokens, and more), plus 8
custom slash commands. Notable examples: the auth domain covered five roles, nineteen
scopes, and eighteen granular permissions; the pricing engine spec mandated 8-decimal
integer arithmetic for all money operations, with floats treated as a test failure; and
payment idempotency was enforced at the database level via a unique constraint specified
before any charge endpoint code existed.

Fontoura is explicit about limitations — one developer, one system, one deadline, no
control group, and no independent quality audit. His structural argument: an AI agent has
no memory between sessions, and the spec is the external memory you give it. Without
specs, "the agent is a fast bricklayer with no blueprint"; with them, "it is a senior engineer
with perfect recall of every decision you made."
