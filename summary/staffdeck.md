---
url: https://github.com/OpenBMB/StaffDeck
title: "StaffDeck: Build, Run, and Govern Enterprise Digital Employees"
author: OpenBMB / ModelBest, THUNLP, Northeastern University joint lab
date_fetched: 2026-10-24
date_published: 2026-07-15
topics:
  - coding-agents-and-frameworks
  - agent-orchestration
---

StaffDeck is an open-source (AGPL-3.0) enterprise platform from OpenBMB/ModelBest and academic partners for building, running, and governing "digital employees" — persistent agent personas with job titles, capability files, knowledge bases, SOPs, and tool access, served over web UI, WeChat/WeCom IM channels, an open REST API, and a CLI. It shipped v0.1 in July 2026 and iterated rapidly to v0.4.0 (multi-employee collaboration) by August.

The backend is a large Python/FastAPI monolith (~97k lines under `backend/app/`) with SQLModel storage, paired with a React/TypeScript workbench. Its distinguishing architecture: (1) **state-machine SOP skills** — natural language is distilled into a typed skill graph (`SkillGraphNode`/`SkillGraphEdge` with slots, conditions, handoff nodes, per-node capability subsets) that an execution engine walks step by step with enforced slot hydration; (2) **document-structure-aware knowledge retrieval** — an ingest pipeline normalizes documents into an "OKF" wiki of typed concepts (source document, section, topic, playbook, business rule) navigated layer-by-layer with citation-excerpt evidence; (3) **a Harness layer** — a registry/executor for validated tool calls, session locks and leases, plus a vendored TypeScript agent harness (`harness-engine/`, DeepSeek's "dsh" everything-is-a-plugin harness) with a Landlock-based Linux sandbox runtime.

Multi-employee operation is a headline feature: a channel adapter layer routes WeChat/WeCom messages to the right digital employee via LLM intent classification with stickiness rules, identity merging binds external chat identities to platform accounts, and **teams** (`backend/app/teams/`) implement a task-bidding market where a team-leader agent assigns work, members bid with HP-deducted scores and rebuttal rounds, and a shared blackboard carries cross-task working memory. Everything streams into an auditable execution trace with human handoff and feedback capture.
