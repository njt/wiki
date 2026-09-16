# Architectural Guardrails for AI-Generated Code

An O'Reilly Radar essay that names a failure mode most teams have felt but few have named: architectural drift, where individually sound agent-generated code collectively pulls a codebase away from decisions the team already made and recorded. Its prescription is a new layer — "engineering governance" — sitting between recorded decisions and the tools that generate, review, and merge code.

---

## The argument in one paragraph

The rework gap widening alongside AI adoption is not primarily a model-quality problem but an organizational memory problem: agents write code that violates architectural decisions because those decisions live in documents the agent never sees and the reviewer never reads. The fix is not better models or more review but a new infrastructure layer — engineering governance — that makes ADRs machine-readable, injects relevant decisions into agent context before code is written, and enforces them deterministically in CI with every verdict traceable to a specific rule and artifact. This is falsifiable: if teams inject structured decisions into agent context and drift persists at the same rate, or if deterministic enforcement proves impractical for genuinely architectural (as opposed to syntactic) rules, the thesis fails. It is also testable against the churn data the author himself cites, which he concedes may partly reflect productive refactoring rather than drift.

## Key quotes

> It's a memory problem. Not the model-internal sense of context window, but the organizational sense.

The sharpest move in the piece: relocating the failure from the model to the organization. Most drift discourse blames the tool; this reframes it as a retrieval and surfacing failure of institutional knowledge — which is a problem you can build infrastructure for.

> These are documents in the shape of configuration.

A precise demolition of Cursor Rules and CLAUDE.md-style files: free text with no precedence rules, no versioning, no lifecycle, no arbitration when rules conflict. The phrase captures why the instinct is right and the resolution is wrong.

> Two probabilistic passes over the same blind spot are not one deterministic pass with sight.

The essay's best one-liner, aimed at LLM-assisted code review. A second model reviewing the first inherits the same missing context, so review-at-scale with models is structurally incapable of catching drift — it can only catch what both models can see.

> Code output scales; review attention does not.

The economic core of the argument, stated in eight words. It is the same asymmetry driving [[The End of Code Review]]: agents can produce many implementations in the time a human assesses one, so asking humans to review harder moves the constraint rather than removing it.

> Probabilistic systems may retrieve or recommend. They shouldn't independently determine an enforcement verdict.

The governance layer's constitutional principle. Every block must reconstruct from artifacts on disk — code, ADR, retrieval log, rule text — so that "why did this fail?" has an answer that isn't "the AI said so." This is what makes the proposal defensible in regulated environments and, more mundanely, in the argument between an engineer and the tool that blocked their merge.

## Critical analysis

The non-obvious contribution is the taxonomy of near-miss solutions. Most writing on this problem either celebrates rule files or demands better models; this essay systematically shows why each existing mechanism — rule files, linters, SCA tools, LLM review, human review — fails at a *different* level, and that the failures compound rather than overlap. The observation that linters cannot enforce "customer writes go through the service API" because it is a semantic decision, not a syntactic property, is the crispest statement of why this gap has persisted.

The determinism principle is the piece's strongest and most underargued claim. It is genuinely important — it separates this proposal from the "AI reviews AI" fashion — but the essay never confronts the hard case: most architectural decisions are not expressible as deterministic rules. "All writes go through the customer service API" is enforceable. "Prefer composition over inheritance" is not, and the boundary between the two is exactly where teams will discover how much of their architecture was actually written down. The essay's own vignette is suspiciously clean: the ADR named a concrete, checkable rule. Real ADR collections are full of rationales, trade-offs, and soft guidance that resist rule extraction. The proposal quietly assumes the corpus can be made rule-shaped without loss.

What is also missing: any engagement with the cost side. A structured corpus with precedence and lifecycle metadata is a maintenance burden — someone must keep it current, resolve conflicts, and retire superseded decisions, which is precisely the labor that failed for the ADR in the opening vignette ("the engineer who wrote the ADR has since left"). Governance layers do not maintain themselves; without a named owner and lifecycle process, the machine-readable corpus becomes a second, staler copy of the documents nobody reads. The essay gestures at this ("audit your ADRs") but treats corpus hygiene as a quarterly task rather than the central ongoing cost.

Finally, the piece is notably vendor-agnostic to a fault — it insists the layer be independent of Cursor, Claude Code, Copilot, and Codex, which is correct but leaves the business question dangling: who builds this, and what does it look like when it is a product rather than a principle?

## Related

- [[Engineering Standards Enforcement at Cloudflare]] — Strengthens this source's thesis with the most detailed working proof: Cloudflare's Codex is exactly the structured, precedence-aware decision corpus this essay calls for, enforced by review agents at merge-blocking scale — evidence the governance layer is buildable, not just desirable.
- [[Principal Drift]] — Nuances this source from the same publisher: where this essay proposes the governance layer, Principal Drift supplies the risk-routing discipline for applying it, and both share the insistence that enforcement verdicts must be reconstructable rather than model-asserted.
- [[A Convention Is Not a Constraint]] — Complicates this source's treatment of rule files: that piece argues conventions only become real when they are mechanically enforced, which supports the essay's critique of CLAUDE.md-style free text while raising the harder question of which architectural decisions can be turned into constraints at all.
- [[The End of Code Review]] — Strengthens the essay's economic premise from the opposite direction: Monperrus argues human review is already indefensible as a quality gate, and this essay's "code output scales; review attention does not" is the same asymmetry — but this source replaces human gating with deterministic enforcement where Monperrus proposes agent-in-the-loop verification.

---
*Sources: [[raw/architectural-guardrails-for-ai-generated-code]], [[summary/architectural-guardrails-for-ai-generated-code]]*
