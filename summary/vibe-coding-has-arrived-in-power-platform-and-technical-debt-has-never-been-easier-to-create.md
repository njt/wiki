---
url: https://arinco.com.au/blog/vibe-coding-has-arrived-in-power-platform-and-technical-debt-has-never-been-easier-to-create/
title: "Vibe Coding Has Arrived in Power Platform, and Technical Debt Has Never Been Easier to Create"
author: Arinco (unnamed Power Platform consultant)
date_fetched: 2026-09-23
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - software-engineering-craft
---

A Power Platform consultant's argument that AI-assisted "vibe coding" makes building software nearly free while leaving every cost of *owning* software intact — and that Power Platform's citizen-developer promise, now supercharged by Copilot-style generation, is a technical-debt machine without governance.

The core move is separating building from operating. AI collapses the cost of the demo ("it works") but not the fourteen production questions that follow: ownership, deployment, identity, credentials, API dependencies, data egress, failure detection, incident routing, rollback, support, testing, documentation, and future change. "It works" has never meant "production-ready," and AI makes that gap wider, not narrower.

The author's prescriptions: the barrier to building should move rather than disappear — experimentation free, production deliberate, with gates at the promotion boundary (ownership, environments, Solution Checker, data policies, rollback, support docs). Governance must become technical — enforced by the platform (Managed Environments, pipelines, DLP policies, Solution Checker enforcement) rather than documented in a 40-page PDF nobody opens at 11 PM. And "Copilot built it" is the new "it worked on my machine" — AI-generated software is still software, inheriting enterprise consequences regardless of author. The developer's job doesn't disappear; it moves up the stack to architecture, security, integration design, and judging whether a five-minute build deserves five years of trust.
