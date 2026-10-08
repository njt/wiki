---
url: https://www.planetgeek.ch/2026/10/06/event-sourcing-fun-with-bi-temporal-timelines/
title: "Event Sourcing: Fun with Bi-Temporal Timelines"
author: Urs Enzler
date_fetched: 2026-10-08
date_published: 2026-10-06
topics:
  - databases-and-data
  - software-engineering-craft
---

Urs Enzler's continuing event sourcing series turns from modelling bi-temporal events to consuming them: what you can actually *do* with the timelines that projecting bi-temporal events produces. A timeline shows how a value changes over time, represented in F# as a discriminated union — either `Existent` with a list of phases (each with a start and optionally a value) or `NonExistent`, with the deliberate distinction that "no data" is a different state from "an empty timeline." Enzler credits Kevlin Henney's "Much Ado About Nothing" talk for why that distinction matters.

The conceptual centre is a reorientation of the default query. In ordinary event sourcing the bread-and-butter operation is "what is the current value?" — but with bi-temporal events, where many events carry future effective dates and historical queries are routine, the most-used function is instead *value at a point in time* (`at`). Supporting operations follow: `atWithStart` returns the value *and* how long it has been unchanged, which matters for calculations that depend on value stability. Slicing a timeline to a range of interest drops unneeded phases to speed later computation, with options controlling exactly how the edges are trimmed. Folding over a timeline drives a state machine across phases — an expense created, accepted, then withdrawn ends in the state *withdrawn* — computing an aggregate state from the phase sequence rather than from a single snapshot.

Implementation notes (in the "Deep dive" sections) show the delegation pattern: a `Timeline` wraps a `PhaseList`, and most operations delegate to it, with F# pipes and partial application keeping the code short. Enzler also mentions he simplified the types based on reader feedback — the series is visibly iterated in public. A closing list teases further `Timeline` module functions (distinct/non-distinct values in a range, `combine2` for merging same-granularity timelines, `choose`, `bind` on nested timelines, and `unzip` for tuple-valued timelines), offered as candidates for future deep dives.

As with earlier parts, this is a practitioner's notebook rather than a tutorial: types shown, behaviour stated, trade-offs named, and readers invited to steer the next instalment.
