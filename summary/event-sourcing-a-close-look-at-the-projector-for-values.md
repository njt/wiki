---
url: https://www.planetgeek.ch/2026/10/12/event-sourcing-a-close-look-at-the-projector-for-values/
title: "Event Sourcing: A Close Look at the Projector for Values"
author: Urs Enzler
date_fetched: 2026-10-14
date_published: 2026-10-12
topics:
  - databases-and-data
  - software-engineering-craft
---

Urs Enzler continues his event sourcing series with the implementation of the *value projector* — the component that folds a stream of uni-temporal (plain) events into a single value representing the latest state. The post is a code walkthrough in F#, presenting the full `ValueProjector` and the helper types around it.

The design centres on a `Projection<'a>` discriminated union with three states: never existed, existing, and *bygone* (deleted, but still carrying the last value). A `ProjectionConfiguration` selects among several projection kinds — create/update/delete events, set-and-remove events, ordered updates, and "streamlined" events from accept/reject workflows. Each kind has its own sort function: create/update/delete are ordered first by application timestamp then by event type, and an explicit update-order exists for the rare case where same-timestamp updates need a deterministic sequence.

The projection itself is a `List.fold` acting as a state machine — invalid transitions such as an update after a delete are simply ignored. Streamlined projections are the most involved: a first pass over accept/reject events builds a mutable dictionary of trigger states (accepted, rejected, undecided), a second pass filters out rejected events, then the same fold as the simple case runs over what remains. The post closes with the public projection functions: single projection, projection with an initial value, and multi-projection overloads keyed by a group function.
