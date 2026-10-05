---
url: https://github.com/nraford7/deeper-research
title: "deeper-research"
author: nraford7
date_fetched: 2026-10-05
date_published: 2026-09
topics:
  - guardrails-and-feedback-loops
  - agent-orchestration
---

A Python + Claude Code/Codex shared skill (~18K lines) that produces a fully-cited "Research Bible" by a six-round retrieval-first pipeline: Round 0 domain scoping → Round 1 Exa retrieval slices plus free OpenAlex/Semantic Scholar academic anchors → a hard evidence gate (exit 22) that refuses thin corpora → synthesis → question-driven deepening → slot-batched integration agents → mechanical citation verification, a number-provenance sweep, and a refute-mode adversary on a different model family.

The core thesis is a rebuild of the author's earlier deep-research (v1): v1 ran five parallel model agents and triangulated on agreement, which ratifies shared hallucinations; v2 fetches evidence before reasoning and binds every claim to a fetched source. Grounding gates added in 2026-09 after a real fabrication incident: evidence retention with pre-marked `[UNVERIFIED]` quantities, a Decimal-exact number-provenance sweep, unverifiable-citation-shape detection, and local KB ingestion with `[kb:slug, year]` cites.

Engineering discipline is unusually high for an LLM pipeline: a crash-safe per-run money ledger that pre-charges worst-case Exa cost (exit 21 on cap breach, never silently retries), fail-closed exit codes throughout, a containment-wrapper requirement for Claude batch runs, and a test suite covering ~40 modules including fixture tests built from the actual fabricated figures of the 2026-09 incident.
