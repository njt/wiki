---
url: https://www.oreilly.com/radar/architectural-guardrails-for-ai-generated-code/
title: "Architectural Guardrails for AI-Generated Code"
date_fetched: 2026-09-16
topics:
  - guardrails-and-feedback-loops
  - specifications-as-the-product
---

An O'Reilly Radar essay naming a specific failure mode of scaled AI-assisted development: architectural drift. A composite vignette shows a clean, tested, agent-written PR that bypasses a banned code path because the architectural decision record (ADR) memorializing the ban was never surfaced to the agent or read by the reviewer. The author argues this is neither hallucination nor model-quality failure but an organizational memory problem — the code was fine; the context was missing.

The piece surveys why existing mechanisms sit at the wrong layer: free-text rule files (Cursor Rules, CLAUDE.md) are "documents in the shape of configuration" with no precedence or lifecycle; linters can't enforce semantic routing decisions; dependency scanners see nothing here; LLM-assisted review shares the first model's blindness; and human review attention doesn't scale with agent output.

The proposed fix is an "engineering governance" layer that holds decisions in a structured corpus with precedence and lifecycle metadata, retrieves them reliably, injects them into agent context before code is written, and enforces them deterministically in CI with verdicts traceable to specific ADRs. The load-bearing principle: probabilistic systems may retrieve and recommend, but enforcement verdicts must reconstruct from artifacts on disk. Leaders are advised to audit their ADRs, choose an enforcement posture deliberately, and not wait for better models.
