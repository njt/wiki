# claude-ctrl

A structured workflow for agentic development implemented as skills and hooks, transforming Claude Code into a deterministic control plane. The core insight: "An instruction that lives only in model context is not a constraint." ClauDEX moves enforcement from prompts to event-based hooks with SQLite-backed policy evaluation, because LLMs are probabilistic systems that require deterministic enforcement boundaries.

---

## Key Quotes

> "An instruction that lives only in model context is not a constraint."

> "LLMs are not deterministic systems with probabilistic quirks. They are probabilistic systems."

## Key Themes

#enforcement #hooks #workflow #determinism #claude-code

The workflow loop (Planner -> Guardian -> Implementer <-> Codex Critique -> Reviewer -> Guardian) is more structured than most agent workflows. The Guardian role is interesting -- it provisions worktrees and lands git changes, acting as the gatekeeper between planning and implementation. The read-only Codex/Gemini CLI providing independent critique is a form of cross-model review.

The SESAP concept (Self-Evaluating Self-Adaptive Programs) -- probabilistic systems constrained to deterministically produce desired outcome ranges -- is the theoretical foundation. It's a useful framing: don't try to make the LLM deterministic, just constrain its output to an acceptable range.

Connects to [[claude-code-config (Trail of Bits)]] (which also uses hooks for enforcement, but with a lighter touch), [[Pre-Commit Lint Checks]] (mechanical enforcement at a different layer), and [[Compound Engineering]] (building trustworthy systems rather than relying on model behavior).

## Critical Analysis

At 1,566 commits this is one of the most actively developed projects in the Claude Code ecosystem, which suggests either deep conviction or endless yak-shaving. The "first-deny-wins" policy engine is a strong pattern -- it's the same approach used in firewalls and IAM systems, applied to agent behavior. The risk is over-engineering: if the enforcement infrastructure is heavier than the actual development work, you've traded one form of complexity for another. The 81.6% Python + 14.6% Shell composition suggests a lot of glue code, which is both necessary and fragile. For security-sensitive work (Trail of Bits' domain), this level of enforcement makes sense. For a personal project, it's probably overkill.

---
*Sources: [[summary/claude-ctrl]]*
*Last updated: 2026-05-14*
