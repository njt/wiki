---
title: "claude-ctrl"
url: https://github.com/juanandresgs/claude-ctrl
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - guardrails-and-feedback-loops
  - agent-orchestration
---

# Claude-Ctrl (ClauDEX v5.0)

Transforms Claude Code into a deterministic control plane by enforcing operational policies through runtime-backed hooks rather than relying solely on prompt-level guidance. Implements workflow governance with SQLite-backed decision-making.

## Core Philosophy
"An instruction that lives only in model context is not a constraint." Moves enforcement to event-based hooks that mechanically deny unsafe paths regardless of what the model remembers.

"LLMs are not deterministic systems with probabilistic quirks. They are probabilistic systems" -- requires deterministic enforcement boundaries rather than contextual suggestions.

## Workflow Loop
Planner -> Guardian (provision) -> Implementer <-> Codex Critique -> Reviewer -> Guardian (land)

Automatic routine landing after reviewer, test, scope, and lease gates pass. Explicit approval reserved for destructive operations.

## Enforcement Model
- Policy engine uses first-deny-wins evaluation
- SQLite runtime serves as system of record
- Hook adapters normalize Claude Code events
- Read-only Codex/Gemini CLI provides independent critique

## Role Structure
- **Planner**: owns requirements, scope, contracts
- **Guardian**: provisions worktrees and lands git changes
- **Implementer**: writes source within leased scope
- **Codex**: CLI-based convergence review (read-only)
- **Reviewer**: technical readiness authority (read-only)

## Key Concept
Self-Evaluating Self-Adaptive Programs (SESAPs): probabilistic systems constrained to deterministically produce desired outcome ranges.

1,566 commits, Python (81.6%), Shell (14.6%), 180 stars.
