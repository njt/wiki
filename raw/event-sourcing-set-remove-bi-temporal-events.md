---
url: https://www.planetgeek.ch/2026/07/07/event-sourcing-set-and-remove-based-bi-temporal-events/
title: "Event Sourcing: set-and-remove-based bi-temporal events"
author: Urs Enzler
date_published: 2026-07-07
date_fetched: 2026-07-18
site: planetgeek.ch
series: Event Sourcing (part 12)
categories: [".NET", "Event Sourcing", "F#"]
---

# Event Sourcing: set-and-remove-based bi-temporal events

Urs Enzler, July 7, 2026

This is part twelve of Enzler's series on event sourcing. It contrasts two kinds of bi-temporal event streams.

## Lifetime (create-update-delete) events

Earlier posts covered the "lifetime" kind, where "a thing is created, updated, and maybe deleted." These use two time axes — effective time (when something takes effect) and application time (when data entered the system). Projection sorts by effective timestamps, with application timestamps used only to resolve ties.

## Set-and-remove-based events

A different timeline type where events don't represent a thing's lifetime but rather "a timeline that specifies which data is relevant at any given time." Examples given:

- Assignment/removal of a calendar to an organizational unit from a given date
- Workday validation rules
- Employee start/stop on a project
- Settings for calculating project activities

## The Override vs. Insert Problem

When an event added later (with a newer application timestamp) has an *earlier* effective timestamp than an existing event, a choice must be made when projecting:

- **Override behavior:** "event C overrides event B" — the new event wins for all effective dates
- **Insert behavior:** "we keep both events so that the value of C is only valid until the effective timestamp of event B"

Enzler notes "both scenarios can be valid," so the system must allow choosing between them.

## Deep Dive: Organizational Calendars

The post walks through an example: a unit's calendar can be "added to an organisational unit per an effective date, and it can also be removed per an effective date." A set event is defined as `Sets (effective, value)` and a remove event as `Removes effective`. The projection is configured with `ProjectionActionConfiguration = SetRemove` and `OverrideBehavior.Override`.

## Technical Details

- The two time axes (effective and application) govern when data is relevant vs. when it was recorded
- Projection behavior must be explicitly configured per event stream
- The override/insert selection directly affects timeline reconstruction
- The next post (July 14) addresses how to reverse events in these timelines

## Key Quotes

- "a timeline which specifies which data is relevant at any given time"
- "a thing is created, updated, and maybe deleted"
- "The events represent the thing's lifetime"
- "both scenarios can be valid"
- "either a later-added event with an earlier effective date is inserted or overrides later events"
