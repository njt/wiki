# Event Sourcing — The Value Projector

Urs Enzler's series instalment walks through the full F# implementation of a `ValueProjector`: the component that folds a stream of plain (uni-temporal) events into a single value representing the latest state. It covers the `Projection` result type, per-kind projection configurations, event sorting rules, and the fold-based state machine at the core.

---

The heart of the design is a three-state result type. A projection is either never-existed, existing, or *bygone* — deleted but still carrying the last value, so queries about deleted data stay answerable. Everything else hangs off a discriminated union of projection kinds: create/update/delete, set-and-remove, ordered updates, and streamlined (accept/reject) events.

> "Invalid events are ignored (e.g. an update after a delete event)."

That single line in the fold's state machine is doing a lot of quiet work: the projector treats an out-of-order event stream as a domain fact to be absorbed rather than an error to be thrown, which is the pragmatic stance most real event-sourced systems end up taking.

> "A rare case, but still a case we have in our domain."

On sorting same-timestamp updates, Enzler resists generalising. The explicit update-order mechanism exists because *his* domain needed it — a good example of configuration being added only when a concrete ordering problem appeared, not speculatively.

> "I use a mutuable dictionary because it is faster than building a map through folding over the events."

The streamlined-projection pass (typo included, verbatim) shows the functional-purist compromise: a local mutable dictionary inside an otherwise pure pipeline, because two-pass filtering over accept/reject triggers is genuinely faster that way. Purism yields where measurement says so.

## Key themes

#concept #pattern — the projection fold as a state machine; bygone-vs-deleted as a modelling choice; deterministic ordering of concurrent updates; accept/reject workflows filtered before projection.

## Analysis

This is the least glamorous and most load-bearing kind of event sourcing content: not the architecture pitch, but the code that has to be right for the pitch to work. The interesting design decisions are all about *ambiguity tolerance* — what happens when two events share an application timestamp, when an update follows a delete, when a trigger's acceptance is still undecided. The answer throughout is "have a deterministic rule and fold anyway," which is exactly the discipline event streams demand.

The bygone state is the quietly important idea. Most projection examples collapse deleted data to `null`; keeping the last value in a `Bygone` case means "show me what this was before it was deleted" becomes free, which pays off the moment anyone asks an audit or support question.

It also pairs naturally with the earlier series instalments in this wiki: [[Event Sourcing — Set-and-Remove Bi-Temporal Events]] defined the set-and-remove and streamlined event kinds this projector consumes, and [[Event Sourcing — Fun with Bi-Temporal Timelines]] handled the bi-temporal cousin of this same problem — this post is the uni-temporal baseline those two complicate. [[Fixing Bugs in Event Sourcing is Hard]] makes a good companion: a deterministic, testable projector fold is precisely what makes Dudycz's corrective-event workflow safe to apply, since replaying the corrected stream gives the same answer every time.

One honest caveat: the post presents code as screenshots, so the wiki's copy carries the prose structure but not the actual F# — the reasoning is legible, the implementation is not.

---
*Sources: [[raw/event-sourcing-a-close-look-at-the-projector-for-values]], [[summary/event-sourcing-a-close-look-at-the-projector-for-values]]*
*Last updated: 2026-10-14*
