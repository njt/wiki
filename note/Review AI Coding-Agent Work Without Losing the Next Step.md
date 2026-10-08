# Review AI Coding-Agent Work Without Losing the Next Step

A checklist-style guide from alterac.ai on structuring the handoff between coding-agent runs — defining checkable results, capturing starting state, demanding evidence with summaries, reviewing diffs against the task, and making the next decision explicit — culminating in a copyable six-field handoff template and a demonstration of how the company's own review workflow keeps the loop inspectable.

---

The premise is quiet and useful: a good handoff answers three questions fast — *what changed, what evidence supports it, what should happen next* — and those answers should live beside each attempt, not in someone's head. Everything else in the piece is machinery for making that true.

> "Replace 'improve the error handling' with observable behavior. For example: 'When the configuration file is missing, show the expected path, exit with a nonzero status, and avoid changing any files.'"

This is the same move the spec-driven camp makes, but compressed into a single sentence a reviewer can grep. The acceptance check is written *before* implementation, and agreeing on allowed files and approval-required actions up front is scope-setting as verification rather than bureaucracy.

> "Avoid turning an intended check into a reported success."

The sharpest line in the piece, and the most agent-shaped failure mode: an agent that *planned* to run integration tests will happily summarize the run as if it happened. The prescribed fix is recording the negative space — "Not run: integration tests require an unavailable test database" — which converts a silent gap into a concrete next step. This is exactly the discipline [[Ways of Checking]] catalogues from the other direction: there, the failure modes; here, the reporting format that exposes them.

> "Separate successful test runs do not establish that the combined result works."

A small but easy-to-miss distributed-systems truth applied to parallel agent attempts: merging two green branches produces a new, untested system. The checklist's insistence on re-running checks on the integrated revision, and on separating acceptance, merge, and deployment approvals, treats agent output with the suspicion real changes deserve.

> "For a fresh attempt, state which assumptions or approaches are being discarded. Preserve useful findings and the reason a previous approach failed."

This is the part most teams skip. A "retry" that throws away the negative result wastes the most expensive artifact the first run produced. The handoff template's *Unknowns* and *Continuation* fields are the mechanism: dated, labelled (in progress / ready for review / blocked / superseded), and verified against the current repository before resuming — because "an old agent process may still be running."

The product note at the end is thin (alterac.ai's own review queue, follow-up work inheriting the prior summary and handoff), but it makes the right point: the loop — "a bounded request, a specific attempt, evidence from the resulting state, and a clear next decision" — is the unit of work, whichever tool you use.

**Themes:** #pattern #concept #tool

## Opinionated take

There is no new idea here — and that is its value. This is the boring, durable layer under all the agent-workflow excitement: define the check, record the state, demand the evidence, decide explicitly. It reads like a checklist distilled from having watched sessions evaporate, and its best contributions are the anti-fictions: report what was *not* run, never let an intended check masquerade as a result, re-verify after integration. The six-field handoff template is genuinely copyable; most "handoff" advice stops at "write a summary." What's missing is any treatment of who reads this under time pressure — a handoff nobody reads is documentation theatre. And the alterac.ai product section shows why vendor checklists should be read with one eyebrow up: the tool is the conclusion as much as the practice.

## Relation to the wiki

- [[What a Useful AI Trace Should Actually Contain]] — strengthens it directly: Iliev argues traces must answer what changed, what evidence, what next; this checklist is the human-side workflow that consumes such a trace, field for field.
- [[Ways of Checking]] — complements it: that page names the failure modes of verification ("checking differently tests it"); this supplies the reporting format (evidence, exit codes, not-run list) that makes those failures visible to the next reviewer.
- [[The End of Code Review]] — nuances it: Monperrus wants agents in the review loop because human review can't scale; this piece quietly assumes a human reviewer with limited attention and shows what a handoff must contain to survive that constraint.
- [[The Plan Is the Program]] — shares the thesis that the durable artifact is the specification-and-decision record, not the code; here the plan extends past the run into the handoff.

---
*Sources: [[raw/review-ai-coding-agent-work-without-losing-the-next-step-10-06]], [[summary/review-ai-coding-agent-work-without-losing-the-next-step-10-06]]*
*Last updated: 2026-10-08*
