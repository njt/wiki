# Fixing Bugs in Event Sourcing is Hard

Oskar Dudycz's concrete, worked-example comparison of fixing the same production bug in a state-based system vs. an event-sourced one, using a hotel reservation's tourist-tax miscalculation. The article is a field manual for the operational side of event sourcing — not why to adopt it, but what it actually feels like to use when something breaks.

---

## Key Quotes

> "In a state-based system, fixing data overwrites the evidence we'd need to check whether the fix worked, which is how one migration ends up repairing the last one."

The article's thesis in one sentence. The state-based version of the hotel bug cascades: migration one fixes tax but reprices rooms at new rates; migration two tries to undo migration one but the original data is gone; each step destroys the inputs the next step needs. Dudycz has been on both ends: "it doesn't get more pleasant."

> "The corrupted value is cityTax, and it's sitting there in plain sight."

The event stream for the same reservation shows exactly what happened: the 14 March `ReservationPriceCalculated` event says cityTax is 98.00 (all seven nights at the new rate), when the correct value is 73.00 (five March nights at the old rate, two April nights at the new). Every input — the rate plan version, the per-night breakdown, the room price — is preserved in the same event. Correction is arithmetic from known data, not archaeological reconstruction.

> "We can name the events the broken build produced."

This is the `buildSha` insight. Dudycz recommends storing the git commit SHA in event metadata. When the broken build is `a4f9c2e`, you query for exactly the 900 events it produced — not 1,240 rows caught by a fuzzy `updated_at` window that includes 340 reservations merely touched during the same week. `correlationId` and `causationId` provide the same precision for diagnosis: tracing a bad value through three handler hops back to the originating request.

> "It's the accountant's move: we don't rub out a ledger line; we post a correcting entry next to it."

The corrective event pattern: instead of editing the wrong `ReservationPriceCalculated`, append a `ReservationPriceCorrected` with the previous total, the corrected total, and a reason. The correction is itself an event, so the next person to open the stream sees the bug, the fix, and the rationale — no institutional memory required. If the correction itself is wrong, the 14 March events are still there, and a second correction is computed from the same inputs as the first. The loop from the state-based version is broken.

> "Her 839.00 is also correct, because she took 42.00 off the rooms to settle an argument she was having on the phone."

The hardest case. A support agent, Anna, manually corrected a reservation with a goodwill discount — a business decision, not a pricing error. A blind bulk recalculation would overwrite her work and generate a third confirmation email to an already-frustrated guest. With the event stream, you can check: if any price-affecting event follows the bad one, skip it for human review. In the mutable model, the bug and the correction are indistinguishable — both are just a number in a column.

> "Ask the business 'how do you fix this today?' There's almost always an existing answer: a form, a manager's approval, a note in the folder. Model that."

The article's practical bottom line: build `CorrectReservationPrice` as a first-class command with permissions, validation, and a mandatory reason. The bulk fix goes through the same command and produces the same events as when Anna runs it from the front desk. Nobody connects to the production database at 23:00.

---

## Key Themes

- **#pattern** — Corrective events as the append-only alternative to in-place data fixes; the accountant's correcting-entry pattern applied to domain events
- **#pattern** — `buildSha` in event metadata as a query primitive for identifying events produced by a specific broken binary
- **#concept** — Event immutability as operational advantage, not ideological purity: keeping the record of the mistake is what lets you check your own work
- **#concept** — Human corrections vs. automated fixes: the structural problem that a bug and a deliberate discount are indistinguishable in mutable state
- **#comparison** — State-based migration cascades vs. event-sourced corrective append: same bug, two architectures, one ends up repairing the last migration
- **#pattern** — Correction-as-feature: building the corrective operation before you need it, routing bulk fixes through the same command path as human operators

---

## Critical Analysis

**The article's strength is its concreteness.** Most event sourcing writing is either theoretical (the log as unifying abstraction) or tooling-focused (how to configure EventStoreDB). Dudycz walks through a specific bug with specific numbers, specific events, and specific failure modes at each step. The dual telling — same bug, both architectures — is what makes the comparison land. You can argue with the conclusions, but you can't argue that the state-based version is a strawman; anyone who's written a production data migration recognizes the cascade.

**The `buildSha` recommendation is cheap and high-leverage.** It's an environment variable and a few lines of metadata construction. In return, you get the ability to query by the thing you actually care about — which commit produced this event — rather than approximating with timestamp windows. This is the kind of operational wisdom that comes from doing the work, not from reading about it. The same applies to `correlationId` and `causationId` for diagnosis.

**What's understated is the organizational cost.** Dudycz mentions that the corrective event means "the next person to open this stream sees that a bug happened, when it was corrected and why." That's a cultural shift as much as a technical one. Many organizations treat bugs as things to be erased, not documented. An event stream that preserves mistakes is an event stream that makes mistakes visible — to operators, to auditors, to regulators. That's a feature in Dudycz's framing, but it's also a liability if your organization rewards hiding errors.

**The "make the correction a feature" advice is the article's most transferable insight.** It doesn't depend on event sourcing. Every system with mutable state needs a corrective operation — a void, a credit, an adjustment — that goes through the same validation and produces the same audit trail as normal operations. Dudycz frames it as event sourcing advice, but the principle applies universally. The horror story he alludes to — "a hotel checkout got stuck on a blocked financial account, which then jammed the entire night audit, with no way out except a migration or a hotfix" — is a failure of correction-as-feature, not a failure of architecture.

**The "clean stream" question is answered well.** If you need a clean event stream later (for a regulator, a migration, a new read model), copy and transform into a new stream; keep the original. Don't edit in place. This is the same pattern as git rebase vs. merge — sometimes you want the messy history, sometimes you want the clean narrative, and the solution is to keep both, not to destroy one.

**What's missing.** The article doesn't address the read-model rebuild after corrective events are appended — in a system with materialized views, you need to rebuild or patch those views after appending corrections. It also doesn't address the case where the bug is in the event *structure* itself (wrong schema, missing field) rather than the event *values*, which is a harder class of problem. And the `buildSha` query assumes a relational or document event store; event stores with different query models (e.g., stream-only access) would need different tooling to achieve the same precision.

---

## Connections

The article sits at the intersection of several threads in the wiki:

- **[[The Log — Unifying Abstraction for Real-Time Data]]** provides the theoretical foundation: Kreps' State Machine Replication Principle is why event immutability works, and his argument that the log is more fundamental than the table is exactly what Dudycz demonstrates operationally. The corrective-event pattern is the log's accountant metaphor made concrete.
- **[[The Log is the Agent]]** extends the same append-only pattern to AI agent architecture. Dudycz's corrective events and Nakajima's fork-and-replay share the same DNA: you can fix forward because the original events are still there. Dudycz's `buildSha` is the operational cousin of Nakajima's content-addressed model caching — both use metadata to make specific events queryable without replaying everything.
- **[[Event Sourcing — Set-and-Remove Bi-Temporal Events]]** addresses the schema/modeling side of event sourcing; Dudycz addresses the operational side. Together they cover what practitioners need: how to model events, and what to do when those events turn out to be wrong.
- **[[Chatto]]** is a concrete event-sourced application that uses the same corrective-event intuition — its message-body/message-metadata split is a variation on the same theme of separating immutable structure from mutable content.
- **[[Bad Data in Production — Response Playbook]]** covers the general incident-response pattern for data quality failures; Dudycz's article is the event-sourcing-specific variant of step four ("fix and verify"), with the key difference that event sourcing makes verification possible because the original inputs survive the fix.

---

*Sources: [[raw/fixing-bugs-in-event-sourcing-is-hard]], [[summary/fixing-bugs-in-event-sourcing-is-hard]]*
*Last updated: 2026-08-07*
