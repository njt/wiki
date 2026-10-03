# Coding Agents Love Decision Records

Duncan Davidson argues that ADRs are the right durable context for coding agents — but only if you explicitly teach the agent how to apply, question, and maintain them. Agents over-adhere to stale rulings and, worse, over-litigate updates; the fix is AGENTS.md conventions that make accepted ADRs binding, superseded ones historical, and Git the only changelog.

---

## Key Quotes

> "When you record intent explicitly, an agent is less likely to mistake an implementation detail for a foundational rule."

The core value proposition, and a subtle one: the failure isn't the agent ignoring rules, it's the agent promoting the wrong things to rules. Recorded intent is a disambiguation device, not just a memory.

> "An agent preserved an outdated storage abstraction across a new feature because an ADR still described it as mandatory. Instead of flagging the mismatch, it added another layer to keep the new requirement technically compatible with the old ruling."

The sharpest observation in the piece: agents adhere to accepted decisions *more rigidly than humans do*. Once a decision enters the context window, it has unusual authority — the agent would rather build a compatibility shim than declare the ruling obsolete. This is exactly the authority-vs-accuracy problem [[Maybe Coding Agents Don't Need a Bigger Memory]] names more generally: memory that isn't known-current is worse than no memory, and an ADR is memory by construction.

> "When you invite an agent to update a decision, a second tendency appears: preserving the deliberation. Every clarification becomes an amendment explaining its own existence at the expense of clarity."

The second failure mode is novel and agent-specific. Humans also bloat documents, but agents do it systematically — appending justification for each edit, restating cross-referenced rules. The result is prose that's still machine-parseable but has lost its value for the human readers Davidson insists ADRs must keep.

> "An agent doesn't need the transcript of every argument. It needs the ruling that governs today and clear permission to stop when the ruling no longer fits."

The closing principle, and a good design rule for any agent-facing artifact: current state plus a license to escalate, not history plus obedience.

## Key Themes

#concept — decision records as agent context
#pattern — binding/proposed/superseded lifecycle encoded in AGENTS.md
#tool — AGENTS.md as the place where documentation governance lives
#pattern — Git history as the changelog, ADR body as current state only

## Analysis

The essay's real contribution is the two-sided failure taxonomy. Most "docs for agents" writing treats documentation as monotone good — write more, the agent reads better. Davidson shows both directions of error: stale authority (agent obeys a dead ruling) and procedural bloat (agent over-maintains the record until it's unreadable). Both stem from the same root — agents treat written-down statements as high-trust inputs — which means the remedy is also one thing: explicit meta-rules about the document's own status.

His concrete AGENTS.md excerpt is unusually practical for this genre: a status taxonomy (accepted / proposed / superseded), a stop-and-discuss rule when task and ADR conflict, one-rule-one-owner with cross-references, and Git as the amendment log. The "stop and propose which side should change" rule is a small guardrail in the [[Guardrails and Feedback Loops]] sense — it converts silent over-adherence into an explicit human decision point, similar in spirit to the tripwires in [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]].

Where I'd push back: the essay assumes ADRs get written and kept current at all. [[Capturing Why Engineering Decisions]] documents how reliably that fails for humans — "every solution requires someone to manually write something. Nobody does." Davidson's agent-maintained ADRs actually answer that objection (the agent writes the update), but it also means the record's accuracy is only as good as the agent's judgment about what counts as a "durable" decision — the same judgment that, per his own first failure mode, it gets wrong in both directions.

This connects to the debt literature: [[Agents and Acquiring Debt]] already prescribes ADRs as the paydown mechanism for comprehension debt, and this essay is the operational manual for making that prescription actually work — including the maintenance discipline that keeps the ADR corpus from becoming its own debt.

---

*Sources: [[raw/coding-agents-love-decision-records]], [[summary/coding-agents-love-decision-records]]*
*Last updated: 2026-10-03*
