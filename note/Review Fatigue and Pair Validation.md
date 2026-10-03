# Review Fatigue and Pair Validation

A practitioner field report (Shift Mag) on why classic PR-based code review collapses under AI-generated code volume, and the team's replacement: **pair planning** (two humans evaluate the agent's proposed options together, before code exists) and **pair validation** (two humans jointly prove the shipped code works, starting from tests). Review moves from a downstream solo reading task to an upstream, paired, conversational act.

---

## The diagnosis

The author's core observation is that the economics of review inverted before anyone noticed: "the more lines of code a PR has, the less likely it is that the changes will get a very detailed review" — and agents maximise the numerator. Four compounding failures:

- **Nobody reviews the prompting.** Engineers now do "prompting and decision-making than coding," but the review gate still inspects only the code — the actual decision surface is unaudited.
- **The reviewer is the first real reader.** The implementer learned the code through the prompting loop; the reviewer starts cold, so a line-by-line pass can take *longer than building it* — a cost sprint planning doesn't budget.
- **Bolt-on AI review agents miss the point.** Throwing an agent at a PR "without any structural and conceptual changes… misses the point of code reviews entirely. The human is no longer in the loop."
- **Context switching rots the queue.** While agents "combobulate," engineers spin up more agents; PRs pile up, and pressure on the pile degrades review quality.

## The prescription: move review upstream, make it social

**Pair planning.** Ask the agent to *propose multiple documented options without code changes*, then think out loud with a colleague — crucially *before reading the agent's output*, because "once you start reading the AI output, it's highly likely your brain will lock in." Alternatives the agent didn't propose become hard to imagine. This is anchoring-avoidance as an explicit ritual.

The economic argument is sharp: the traditional flow plans the implementation **twice** — author first, reviewer in more detail second — and a disagreement sends author and agent back to the start, burning all tokens on the wrong path. Pairing at planning time means "it's more likely you will choose the best implementation option and reach your goal in minimal time and tokens spent." Disposable-code scenarios are treated as a cost centre to be designed out, not a fact of life.

**Structural prompt committing.** A per-task folder — `prompt.md` (append-only prompt log), `context.md`, `plan.md`, `summary.md` — preserves the *why* and *what-alternatives* that commits destroy. Temporary scaffolding, deleted after; real architectural decisions graduate to ADRs.

**Pair validation.** Deliberately not "pair review": validation means *proving* the code works, starting from the tests ("Are they proving the new code is working? Are they covering all use cases?"). Two people walk the changes together, resolving concerns in conversation instead of leaving async comments "someone should check between three different context-switching sessions." The load-bearing premise: "Since AI is generating most code, we are all validators more than implementers. Let's not validate twice."

**Teamwork begins before the task is started** — allocate equal sprint capacity to implementer and reviewer, so "just review this quickly please" can't happen. The author is honest about limits: open source, time-zone-distributed teams, and contractor engagements don't fit, and he expects different frameworks to emerge for them.

## Key quotes

> "Why do we expect standard code reviews will still work when AI generates much more code volume?"

The whole essay in one question — the review ritual is being preserved by inertia while its inputs changed by orders of magnitude.

> "If they go through everything line by line, it probably takes them longer than it took the original developer."

The strongest of the four failure points, because it's measurable and almost never reflected in capacity planning.

> "Since AI is generating most code, we are all validators more than implementers. Let's not validate twice."

A role-identity claim, not just a process claim — and the reason the final stage is *validation* (prove it works) rather than *review* (read it).

> "Once you start reading the AI output, it's highly likely your brain will lock in."

The anchoring insight is the essay's best practical detail: the order of operations (human thinking first, agent output second) is the intervention.

## Themes

#concept #pattern

## Opinionated take

This is the most concrete *social* answer yet to the review bottleneck. Most of the wiki's review-rethinking sources reach for automation — reviewer lanes, judge agents, verification budgets. This essay reaches in the opposite direction: more humans, earlier, together. That's either a regression to pre-PR pairing or the correct insight that review was never really about reading diffs — it was about shared understanding, and diffs were just the medium. The essay bets on the latter, and the anchoring argument gives the bet real force: you cannot review a decision you didn't participate in.

The weaknesses are the ones the author half-admits. Pair planning assumes co-located or at least synchronously available colleagues — the exact environments he excludes are where most software gets written. And "structural prompt committing" is a discipline-heavy convention that will decay under deadline pressure unless tooling enforces it. There's also an unexamined tension: pair validation starting "from the tests" inherits the tests-are-the-spec assumption that other sources (mutation testing studies, TDD-in-the-loop experiments) have already complicated. Still, as a team-level process change requiring zero new tooling, it's unusually actionable.

The deepest implication is for [[The End of Code Review]] debate: this essay doesn't abolish review, it *relocates* it — from the diff to the plan, and from async comments to conversation. If agents write the code, the scarce human resource is shared intent, and that's cheapest to establish before the first token is generated.

## Related pages

- [[The End of Code Review]] — Monperrus argues mandatory human review is indefensible and review should become agent-in-the-loop verification; this source complicates that by making the human loop *stronger* (paired, upstream) rather than weaker — the disagreement is over where review lives, not whether it dies.
- [[Agentic Code Review]] — Osmani's field guide documents the same bottleneck (longer reviews, churn) but resolves it with human-on-the-loop automation practices; this essay is the fully-human counterpoint from the same diagnosis.
- [[Human-in-the-Loop is Tired]] — Summers names supervision fatigue as the psychological cost of LLM-assisted programming; this source is a team-structural response to exactly that fatigue, redistributing the load rather than asking individuals to endure it.
- [[Understand to Participate]] — Litt argues understanding is the prerequisite for remaining an active collaborator rather than a rubber stamp; pair planning operationalises that by forcing shared understanding before generation, not after.

---
*Sources: [[raw/review-fatigue-12276]], [[summary/review-fatigue-12276]]*
*Last updated: 2026-10-03*
