# The New Unit of Work for Document Collaboration

Lexifina proposes that the agent run — a prompt that writes, lints, compiles and tests atomically — is a new unit of work finer than a commit, and builds a provenance system where author, approver and reason attach to the character itself, applied to legal document drafting via agent-proposed tracked changes.

---

The essay starts from a version-control observation: code never had multiplayer because half-typed code doesn't compile, so collaboration collapsed into commits, and commits keep state but lose reasoning. The agent run is a finer checkpoint — "much smaller than a commit, and much bigger than a keystroke" — and crucially the first checkpoint that carries its own explanation: a prompt is written *before* the work and the run is recorded, not reconstructed from memory after the fact like a commit message.

The application domain is contracts, which Lexifina treats as documents that "behave partly like code": defined terms and cross-references break under half-edits. Lawyers already live in a commit-like rhythm — the emailed redline is the commit, the comparison the diff, the reply the review. Agents enter this by proposing tracked changes, one run = one set of proposals, with per-character attribution:

> Git separates author from committer, but the approver lives in the pull request, never in blame. Here the author, the approver and the reason sit on the same character.

That is the strongest claim in the piece, and it is a real gap in git. Blame names the last commit; it cannot show who accepted an agent's proposal or why. The article cites research that most AI-generated pull requests get no recorded human review, and argues that recording a decision per proposal makes oversight measurable rather than aspirational.

The accountability table is the practical core: a person is measured in keystrokes grouped into ≤30-second active slices; a person-operated agent in one run per prompt with tokens and cost of every model call; a subagent additionally per instruction and per proposed tracked change, sub-runs counted against their parent run. Writes by the server carry *who they were made for*, so an agent's proposal and a human accepting a batch never collapse into one generic "AI".

#concept #pattern

## Critical take

The framing is genuinely useful — "the prompt as checkpoint" is a better unit for agent provenance than either commit or session — but the essay is a product pitch wearing a theory hat, and two weaknesses show. First, it admits a contract has no compiler, so the atomicity that makes the run checkpoint-like degrades to warn-level heuristics for exactly the domain it champions; the analogy is strongest for code, weakest where it is applied. Second, per-character lineage under cut/move/edit is confessed to be a "multi-layered best attempt" — which is where most real disputes would land.

Still, the critique of git is fair and the design implication — attribution richer than blame, decisions recorded at proposal granularity — connects directly to the broader argument that agent work needs its own trace format, not a retrofitted commit message.

## Related pages

- [[DeltaDB]] — this piece cites Nathan Sobo's "Software Is Made Between Commits" directly and extends the same thesis to documents; where Sobo grounds the between-commits space in a database, Lexifina grounds it in legal redlines and per-character blame.
- [[What a Useful AI Trace Should Actually Contain]] — both argue agent work needs a purpose-built provenance record; Lexifina's run/tokens/cost table is a concrete schema answering that piece's question.
- [[Who Owns the Code Claude Wrote]] — Lexifina's server-side "made for whom" attribution is a stronger technical answer to the authorship question that page raises from policy and licence angles.
- [[AI-Written Change Descriptions]] — the observation that a prompt is written before the work while a commit message is reconstructed afterwards complicates that page's treatment of AI change descriptions: here the explanation is native to the unit, not generated post hoc.

---
*Sources: [[raw/the-new-unit-of-work]], [[summary/the-new-unit-of-work]]*
*Last updated: 2026-10-14*
