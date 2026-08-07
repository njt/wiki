---
url: https://event-driven.io/en/fixing-bugs-in-event-sourcing-is-hard/
title: "Fixing bugs in Event Sourcing is hard, for real?"
author: Oskar Dudycz
date_fetched: 2026-08-07
---

# Fixing bugs in Event Sourcing is hard, for real?

Oskar Dudycz walks through the same production bug — a tourist-tax calculation error in a hotel reservation system — implemented twice: once in a state-based system where fixing data overwrites the evidence, and once in an event-sourced system where the original events survive every correction attempt. The article is a practical field manual for the operational side of event sourcing that most introductory material skips: what happens on Tuesday morning when someone from support says the numbers look wrong.

The state-based version is a cascade failure: the initial bug miscalculates tax for 900 reservations; the migration to fix it reprices 400 pre-bug reservations at new rates (because the original inputs are gone); a second migration must reconstruct booking dates from confirmation emails to undo the first migration's damage. Each fix destroys the evidence the next fix needs. Patching the system afterward — JSON columns, price history tables, `corrected_by` markers — amounts to building a partial event log, one column at a time, under time pressure.

The event-sourced version has three structural advantages. First, the original `ReservationPriceCalculated` event still carries all its inputs (per-night breakdown, rate plan version), so correction is calculation from known data, not reconstruction from guesswork. Second, a `buildSha` in event metadata lets you query for exactly the events the broken binary produced — 900 events, not 1,240 rows caught by a fuzzy `updated_at` window. Third, a corrective event (`ReservationPriceCorrected`) is an accountant's correcting entry: append, don't overwrite. The next person opening the stream sees that a bug happened, when it was corrected, and why.

The hardest case is when a customer has already been handled — a support agent agreed to a goodwill discount that a blind recalculation would silently undo. In the state-based model, a bug and a deliberate correction look identical: a number in a column. With the event stream, you can check whether any price-affecting event follows the bad one and skip that reservation for human review.

Dudycz recommends building the correction operation as a first-class command (`CorrectReservationPrice`) before you need it, with permissions, validation, and a mandatory reason — so the bulk fix goes through the same rules as a human operator, and nobody connects to production at 23:00.
