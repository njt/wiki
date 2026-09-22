# hipaudit — Written Debt Made Queryable

The README of hipEngine's `audit/` subsystem describes `hipaudit`, a Python CLI that turns a codebase's already-written-down debt — refactor-ledger headings, environment flags, kernel sources, campaign records, worklog entries — into a queryable, gated workflow for human-and-agent crews. Its thesis inverts the usual debt-tool pitch: discovery was never the problem (agents had documented 396 REFACTOR headings, 10,590 worklog entries, 747 flags); the missing layer was workability. The design is a stack of small durable mechanisms — a standing inventory of triageable rows, a findings queue whose checks name their own closing edit, a decision store that survives text rewrites via match hints and reports its own staleness, and a budget gate that can only ratchet down — with read-only triage agents structurally split from code-editing fix agents.

---

## Key Quotes

> "The problem was never discovery. None of it was queryable, so nothing could be worked through, and new debt landed faster than old debt retired."

The inversion that makes this document interesting. Almost every debt tool sells *finding* more debt — more linters, more scanners, more dashboards. This one starts from the opposite premise: the corpus already existed, exhaustively, and it was inert. A 10,590-entry worklog nobody can work through is not documentation; it's a landfill with an index. The bottleneck wasn't knowledge, it was the absence of a queryable surface that turns knowledge into units of work.

> "Never record a decision you did not verify. If the evidence does not settle it, leave the row open and say so. An untriaged row is honest; a fabricated disposition is not."

The core honesty rule, and it is written for agents first. An agent handed a triage assignment has every incentive to empty the queue, and an empty queue with fabricated dispositions is worse than a full one — it's a lie that looks like progress. "An untriaged row is honest" is a remarkable inversion of the usual backlog-guilt framing: leaving work open *with evidence of having looked* is the virtuous outcome.

> "A row is evidence, not a verdict. An extractor reports observations — 'no read site under `hipengine/`', 'names 7 paths, 3 no longer exist' — and never a conclusion."

The signal/verdict separation is the epistemological spine of the whole tool. Extractors see what greps see; deciding is a separate, evidence-checked act. "The row's signals are leads, not conclusions — the extractor only sees what greps see" keeps the mechanical layer honest about its own blindness, which is exactly what a triage agent needs to be told before it starts confidently closing rows.

> "The budget is the anti-laziness gate. `budget.json` records the untriaged count per kind. `check` fails when a count rises, so new debt cannot land without being triaged. **Never raise the budget to get past the gate.** Lowering it is the point."

A ratchet applied to triage debt rather than code patterns: the ceiling is untriaged counts, the automatic direction is down, and raising it is "a deliberate act needing a recorded reason" that `refresh` never performs. This is the [[Ratchets in Software Development]] mechanism generalized from counting deprecated patterns to counting undecided work — and it closes the loophole that kills most backlogs, where new debt lands faster than old debt retires.

> "`worklog` needs this because its population grows with every session: gating all of it would tie the ceiling to the rate of work instead of to the backlog awaiting a decision, and a permanently red gate is a gate nobody reads."

The `select` block is the mature addition to the ratchet idea. qntm's ratchet has no answer for a metric that grows with ordinary productive work; this design does — narrow the gate to the rows that actually need a decision, keep the rest visible but out of the ceiling. The quoted sentence is Goodhart-aware engineering: the designers know a gate that always fires trains people to ignore gates.

> "Triage decides; it does not fix. The fix is its own unit, with its own tests and commit. The exception is `--resolved` when you confirm the work was already done." — and, on the agent split: "keeping them apart is what stops a cleanup commit from also changing behaviour."

Separation of powers, stated as workflow design. The read-only triage agent and the code-editing fix agent share one decision store but never one commit. This is the builder/reviewer separation applied inside a single repo's cleanup loop, and it targets a specific, recognizable failure: the "tiny cleanup" PR that also refactors the auth path.

> "The inventory says what is *catalogued*, not what is *true*. A row with no signals is not thereby healthy, and a row with several is not thereby debt. The counts in any report are a floor. Nothing here replaces reading the code."

The closing caveat is load-bearing, not boilerplate. The entire apparatus — extractors, budgets, gates — is explicitly subordinated to reading the code. A tool that spends most of its README on durable decision machinery and ends by denying its own authority has its epistemic hierarchy the right way round.

## Key Themes

#tool #pattern #concept #guardrails #technical-debt

- **#tool hipaudit** — a single-file CLI over a small directory layout (`hipaudit/core.py`, `inventory/`, `checks/`, `report.py`) whose generated state (`inventory/*.json`, `findings/*.json`) is committed precisely so the diff between two scans is reviewable. Hermetic unittest suite included.
- **#pattern Signals are observations, verdicts are decisions** — extractor rows carry `evidence` (facts), `signals` (observations), and required `hints` (anchor, refs, tokens) for identity across rewrites; "dead kernel" is a verdict and belongs in triage. Checks must satisfy "precision over recall": "a noisy check gets ignored, which is worse than an absent one."
- **#pattern Durable decisions under entropy** — decisions in `triage/*.jsonl` record a hash of the evidence they were made against; content-derived ids plus match hints re-attach a decision when its row's text moves (a row that already has a decision is never claimed by a rebind); moved evidence returns the row `stale`; `--expires` returns conditional calls on a schedule; orphans are listed, never silently dropped.
- **#concept The downward-only budget** — the anti-laziness gate: untriaged counts per kind, `check` must exit 0, automatic movement only downward, with the `select` block as the escape hatch that keeps the gate honest instead of permanently red.
- **#concept Rules compiled into tags** — `GATE-CATCH22`, `LOST-OPT`, and `EXACTNESS-REJECT` encode prose rules from `AGENTS.md` and `docs/OPTIMIZATION.md` as executable debt tags, because "these are the failure shapes this project actually has." The wiki's "an instruction in context is not a constraint" thesis, applied to a repo's own rulebook.
- **#pattern One workflow per session** — "Gathering, triaging, and fixing are different kinds of work with different outputs, and mixing them produces a commit nobody can review." Scope runs by group or signal, never "triage everything."

## Critical Analysis

**The premise is the contribution.** Every other debt-management source in this wiki starts from the assumption that debt is invisible and needs better detection. This document starts from the opposite: detection was solved so thoroughly — by agents, over months — that the debt corpus itself became the problem. That's the dark-factory trajectory taken to its logical end: agents produce documentation at machine speed, and the human-scale bottleneck moves to working through it. The tool's answer is not smarter summarization but *workability*: ids that survive rewrites, decisions that outlive the session, a queue that names its own closing edit.

**This is really a design doc for durable state, not for linting.** Look at where the engineering effort concentrates: content-derived row ids, match hints, similarity-scored rebinding, evidence hashes, staleness reporting, expiry, orphan listing, a never-stolen decision invariant. Each mechanism defends against a specific way remembered conclusions rot: text drift, evidence drift, silent calcification, vanished referents. That is a memory architecture for decisions — which is why it pairs so naturally with the wiki's comprehension-debt pages rather than its linter pages.

**The honesty rules are anti-hallucination process, and their enforcement has a hole.** "Never record a decision you did not verify" and "a fabricated disposition is not [honest]" are exactly the rules you want a triage agent to hold. But the gate counts *untriaged* rows; it cannot detect a fabricated disposition, because a fabricated disposition looks identical to a verified one in `triage/*.jsonl`. The trust anchor is the evidence-carrying note — "a note that restates the tag is not a note" — which is only as good as whoever reads it, human or agent. The design knows this ("needs a human" closes every refresh) but never names the gap: the one failure mode the gate can't see is the one agents are most prone to.

**The budget gate is qntm's ratchet grown up.** Same core mechanism — count, compare, refuse to rise — but with two additions the original lacked: the automatic direction is explicitly downward (cleanup shows up as a number), and the `select` block handles the case where the counted population grows with healthy work. "A permanently red gate is a gate nobody reads" is the sentence every alerting system should be required to contain.

**The bespoke-ness is both the point and the limit.** The extractors and checks are unmistakably hipEngine's: `HIPENGINE_*` flags, ROCm test guards, `__global__` kernel entry points, campaign candidates rejected on an exactness bar. The *pattern* (inventory + findings + durable decisions + downward gate + agent split) is portable to any repo; the *code* is not. Anyone hoping to lift this wholesale will be writing their own extractors — which the README anticipates with a registration API and the instruction to run `budget` after adding one.

**The premise quietly contains the tool's own risk.** 151 one-off `*audit*` scripts, written per campaign and never consolidated, are the problem statement — and `audit/` is script 152. What distinguishes it is exactly what it enforces on itself: committed diffable state, a downward-only budget, contract tests, and a rule ("precision over recall") aimed at its own noise. Whether that holds is precisely what its own budget graph would show. The tool is its own first test subject.

## Relation to the Wiki

- **[[Ratchets in Software Development]]** — strengthens it: the budget gate is the ratchet generalized from counting deprecated code patterns to counting untriaged debt, with the automatic direction pinned downward and the `select` block answering the one question qntm's version leaves open (what to do when the counted population grows with healthy work).
- **[[Structural Backpressure Beats Smarter Agents]]** — strengthens it with a different substrate: where Brooks pushes invariants into type systems, hipaudit pushes them into a committed, gated store — `check` must exit 0, the budget only moves down automatically, and `GATE-CATCH22` even enforces that a restriction with no command to lift it is itself a bug.
- **[[Agents and Acquiring Debt]]** — nuances its fix: bl00cyb prescribes recording decisions in ADRs at decision time; hipaudit is that prescription built out with the machinery ADRs lack — evidence hashes, rebinding across rewrites, staleness, and expiry, so a decision made against changed facts "comes back" instead of calcifying.
- **[[Silently Resolved Ambiguity Is Comprehension Debt of Intent]]** — complicates it constructively: bl00cyb's debt entry "with no signature" is what hipaudit refuses to produce — every decision requires a note carrying `file:line`, a command and its output, or a commit, making the signature and the evidence inseparable.

Related: [[Guardrails and Feedback Loops]], [[Migrations — The Sole Scalable Fix to Tech Debt]], [[Backlog Hierarchy Problem]], [[Pre-Commit Lint Checks]]

---
*Sources: [[raw/hipengine-audit-readme-for-the-hipaudit-debt-management-subsystem]], [[summary/hipengine-audit-readme-for-the-hipaudit-debt-management-subsystem]]*
*Last updated: 2026-09-22*
