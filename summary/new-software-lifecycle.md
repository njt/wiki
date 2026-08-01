---
url: https://www.oreilly.com/radar/the-new-software-lifecycle/
title: "The New Software Lifecycle"
author: Addy Osmani
date_fetched: 2026-07-18
date_published: 2026-07-15
---

Addy Osmani's essay on how AI coding agents reshape the software development lifecycle, drawing from a Google whitepaper co-authored with Shubham Saboo and Sokratis Kartakis. Originally published on Osmani's blog and republished on O'Reilly Radar.

The central framework: an agent is roughly 10% model and 90% harness — the harness being instructions, tools, MCP servers, sandboxes, orchestration logic, and observability. Osmani argues most agent failures are configuration failures, which is encouraging because configuration is fixable without waiting for better models.

Context engineering is the critical financial lever. Static context (system instructions, rule files, global memory) loads every turn and costs on every call. Dynamic context (skills triggered by task matching, tool results, RAG documents) loads on demand. The boundary between them should be treated as an architectural decision, reviewed in PRs and versioned like code. Agent skills with progressive disclosure — minimal metadata at startup, full instructions on match, heavy reference material only when needed — is the scaling strategy.

Verification is what separates vibe coding from engineering. Tests cover deterministic parts; evals cover non-deterministic ones, split into output evaluation (is the result correct?) and trajectory evaluation (was the reasoning path sound?). The advice for leaders: "Set the bar at the eval, not the demo."

AI compresses the lifecycle unevenly. Implementation drops from weeks to hours, but requirements, architecture, and verification remain slow because they involve judgment. Architecture is "the most stubbornly human phase." Specification quality becomes the new bottleneck, and verification moves to the center of the process. Maintenance is the most underrated phase — code previously too risky to touch can now be read, refactored, and modernized by an agent.

The economics invert conventional intuition: vibe coding is cheap upfront but expensive to run (token burn, maintenance tax, security cleanup). Agentic engineering costs more upfront (schemas, tests, structured context) but less per feature afterward. Routing hard reasoning to large models and routine work to small, cheap ones is a key cost lever.

The prototype is becoming the production agent — the same terminal workflow that produced throwaway scripts can now yield deployable agents with persistent memory, scoped permissions, and eval coverage. Osmani distinguishes two daily modes: the conductor (real-time, in-IDE) and the orchestrator (async goal-handoff), calling the shift from one to the other "a skills shift before it's a tooling one."

Adoption numbers as of early 2026: 85% of professional developers use AI coding agents regularly, 51% daily, and roughly 41% of new code is AI-generated. The essay closes with the observation that AI amplifies whatever engineering culture it lands in, and that generation is largely solved — specification and verification are the capabilities worth developing.
