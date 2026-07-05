---
url: https://arxiv.org/html/2605.06445v1
title: "Constraint Decay: The Fragility of LLM Agents in Backend Code Generation"
author: Francesco Dente (EURECOM), Dario Satriani (University of Basilicata), Paolo Papotti (EURECOM)
date_fetched: 2026-07-05
date_published: 2026-05-07
---

Full paper content fetched from arXiv. Original URL: https://arxiv.org/html/2605.06445v1
License: CC BY-NC-SA 4.0

## Abstract

The paper investigates how LLM agents perform in autonomous backend code generation when strict structural constraints are imposed. The authors find that while agents excel under loose specifications, performance degrades substantially when architectural patterns, database backends, and ORM layers are mandated. They term this phenomenon **constraint decay**.

## Key Findings

- Among eight capable configurations (L0 A% > 50%), A% dropped by an average of **30 percentage points** from L0 to L3 (a 40% relative loss)
- Most resilient: OpenHands + MiniMax-M2.5 (dropped 17 pp). Worst: OpenHands + Qwen3-Coder-Next (lost 45 pp)
- Even the strongest L3 configuration reached 78.6% A% but only 8.3% pass@1 — agents still lack cross-file consistency
- PostgreSQL caused the largest drop (−19.3 pp), SQLite (−14.3 pp), Clean Architecture (−9.1 pp). ORM effects were small
- Framework choice is a major sensitivity axis: Express (51.4%), Koa (50.7%), Flask (49.3%) vs Django (25.4%), FastAPI (24.2%), Hono (18.5%)
- Data-layer defects (incorrect query logic + DB/ORM runtime errors) drive ~45% of logic failures across both tested models
- Logic errors dominate at ~71% of failures; server startup failures account for 12–21%

## Methodology

80 generation tasks (8 frameworks × 10 constraint combinations across 4 levels L0–L3). Each task provides an OpenAPI 3.0 spec from the RealWorld Conduit API (19 CRUD operations, 5 resource groups), structural constraints, mandatory files, and a pipeline description.

Two agent scaffolds (Mini-SWE-Agent, OpenHands) and 7 models across 4 tiers: Devstral-Small (24B), Qwen3-Coder-Next (80B), Qwen3-235B-Instruct, MiniMax-M2.5, Kimi-K2.5, GPT-5-mini, GPT-5.2.

Three constraint dimensions: architectural pattern (Clean Architecture), database backend (PostgreSQL/SQLite), and ORM integration (SQLAlchemy/Sequelize).

Isolated Docker execution with a unified HTTP test suite of 32 requests, 291 assertions, plus static verifiers for structural compliance.

Total evaluation consumed ~5 billion tokens.
