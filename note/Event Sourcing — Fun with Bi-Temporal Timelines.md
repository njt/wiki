# Event Sourcing — Fun with Bi-Temporal Timelines

Urs Enzler's next instalment in his planetgeek.ch event sourcing series moves from *modelling* bi-temporal events to *querying* them: the post defines the timeline data structure his system projects events into, then walks through the operations that make timelines useful — point-in-time lookup, slicing, and folding over phases to drive a state machine. The pivot of the piece is that in a bi-temporal system the default query inverts: not "what is the value now?" but "what was the value at *at*?"

---

## Key Quotes

> "In our system, many events have an effective date in the future. On the other hand, we often need to query past data. So, *at* is the normal case."

The quiet inversion at the heart of bi-temporal design. Anyone who has built event-sourced systems reaches for "current value" reflexively; Enzler's domain (insurance-style effective-dated data) makes point-in-time the common path and "now" just the special case `at = now`. This reframes bi-temporality not as an exotic add-on but as a change to which query is primary.

> "To represent the fact that we don't have any data, we use `NonExistent`, not an `Existent` timeline with no phases. These two states are not equal."

A type-level answer to a semantic question. Empty-means-no-data and there-is-no-data are different answers, and collapsing them is exactly the kind of ambiguity that surfaces later as a bug. The nod to Kevlin Henney's "Much Ado About Nothing" places the piece in the long lineage of null-semantics arguments, but the F# discriminated union makes the distinction *unrepresentable to confuse* rather than merely discouraged.

> "We often also need to know how long this value was unchanged, so we can use the `atWithStart` function."

The detail that separates a demo from a production library. "Value at t" alone is insufficient for business calculations that care about duration at a rate — you need the phase's start too. Exposing `atWithStart` alongside `at` shows the API grew from real consumer demand, not from a textbook.

> "I cleaned things up based on insights I had while writing this post series and your comments."

The series as a public refactoring log. The types got *simpler* as the series progressed — the opposite of the usual accretion — which is a nice data point that writing-as-teaching can drive simplification rather than elaboration.

---

## Key Themes

#concept — Bi-temporality: knowledge-time vs effective-time, and timelines as the projection that makes both queryable.
#pattern — Discriminated unions over null-adjacency: `Existent`/`NonExistent` as a total model of possible answers.
#tool — F# and the `Timeline`/`PhaseList` delegation structure; pipes and partial application as the glue.

---

## Analysis

This is the weakest entry in Enzler's series so far, but by the standard of *what it is trying to be* — a practitioner's utility-belt chapter, not a manifesto — it succeeds. The genuinely valuable idea is small and sharp: once you commit to bi-temporal events, your query layer's centre of gravity moves from "current state" to "state at t," and every downstream API (slicing, folds, phase starts) falls out of that shift. The fold-over-phases pattern — replay a state machine across timeline phases to get an aggregate outcome like *withdrawn* — is a clean answer to a question that otherwise gets solved with ad-hoc re-derivation code in every consumer.

The honest criticism: the post leans on the series' earlier instalments and omits the actual code bodies ("Deep dive" sections reference code that the fetch doesn't render as images), so it reads more like release notes for a library than a self-contained explanation. The `NonExistent` vs empty-`Existent` distinction is asserted with a pointer to an unpublished talk rather than argued — the reader must supply the motivating failure case (presumably: confusing "no data yet" with "data was removed" in a system where future effective dates are routine). And `combine2`'s "same granularity" constraint is mentioned but not defended, which is where the interesting design tension actually lives.

Still, taken with the rest of the series, the incremental, feedback-driven simplification is itself instructive: this is what designing a data model in public looks like, and the willingness to shrink types between parts is rarer than the willingness to add them.

## Relation to the wiki

- [[Event Sourcing — Set-and-Remove Bi-Temporal Events]] — the direct predecessor in the same series: where part twelve defined how late-arriving events land in the stream (override vs insert), this post shows the payoff — the timelines that projection produces and the query operations they enable. Reading them together turns two isolated technique notes into a coherent pipeline.
- [[Fixing Bugs in Event Sourcing is Hard]] — Dudycz's operational field manual is the debugging complement: his correcting-event pattern and Enzler's point-in-time queries are two halves of the same trust-in-history posture, and the bi-temporal timeline makes Dudycz's "compute the correction from preserved inputs" even more mechanical.
- [[Domain Events Are Not Your Public API]] — Comartin's boundary argument nuances this: a timeline is an *internal* query shape, and this post is a good example of why exposing it directly to consumers would recreate the coupling Comartin warns about.
- [[Databases and Data]] — extends the topic's coverage of temporal data modelling beyond storage-engine concerns into the query layer, adjacent to [[ARIES — Write-Ahead Logging Recovery]]'s history-preserving instinct.

---
*Sources: [[raw/event-sourcing-fun-with-bi-temporal-timelines]], [[summary/event-sourcing-fun-with-bi-temporal-timelines]]*
*Last updated: 2026-10-08*
