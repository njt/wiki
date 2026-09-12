---
url: https://www.planetgeek.ch/2026/07/07/event-sourcing-set-and-remove-based-bi-temporal-events/
title: "Event Sourcing: set-and-remove-based bi-temporal events"
author: Urs Enzler
date_fetched: 2026-07-18
date_published: 2026-07-07
topics:
  - software-engineering-craft
---

Part twelve of Enzler's event sourcing series. It contrasts two kinds of
bi-temporal event streams: the "lifetime" kind (create-update-delete, covered in
earlier posts) and "set-and-remove-based" events, where the timeline specifies
which data is relevant at any given moment — like assigning or removing a
calendar to an organizational unit, or an employee starting and stopping on a
project.

Both kinds use two time axes: effective time (when something takes effect) and
application time (when data was recorded). The key design choice is what happens
when an event with a later application timestamp has an *earlier* effective
timestamp. The system must support two behaviors: **override** (the new event
wins for all effective dates) or **insert** (the new event applies only until
the next event's effective date). Enzler argues both are valid, so projection
behavior must be explicitly configured per stream.

The post walks through organizational calendars as a detailed example, defining
set events (`Sets(effective, value)`) and remove events (`Removes effective`),
with projection configured as `SetRemove` + `OverrideBehavior.Override`. The
next post in the series covers reversing events in these timelines.
