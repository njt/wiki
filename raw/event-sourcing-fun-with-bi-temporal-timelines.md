---
url: https://www.planetgeek.ch/2026/10/06/event-sourcing-fun-with-bi-temporal-timelines/
date_fetched: 2026-10-08
---

In earlier parts of this series on event sourcing, we have seen that we can project bi-temporal events (events with two timestamps: when the system knew about the event and when the event takes effect) into so-called timelines. A timeline shows a value over time. In this post, we look at some exemplary use cases these timelines enable.

## What a timeline is

A timeline shows how a value changes over time:

A timeline in our system is a discriminated union with two values. It is either

- `Existent`with a list of phases, and every phase having a start and either having an associated value or representing a phase without a value; or
- `NonExistent`.

We can get a `NonExistent` timeline when we ask the system for some non-existent data. To represent the fact that we don’t have any data, we use `NonExistent`, not an `Existent` timeline with no phases. These two states are not equal. *A great talk on why this distinction matters is Kevlin Henney’s “Much ado about nothing” (no recording available yet).*

Feel free to skip the *Deep Dive* sections if you are only interested in the conceptual part of this post.

### Deep dive

These are the F# types to represent a timeline:

*Long-time readers may notice this code looks a bit simpler than in older posts. Yes, I cleaned things up based on insights I had while writing this post series and your comments. Thanks for that.*

## The value at a point in time

Probably the most-used functionality on timelines is getting the value at a specific point in time.

If you are new to bi-temporal events, you might be surprised that the most-used functionality isn’t getting the current value (what we typically do in normal event sourcing). In our system, many events have an effective date in the future. On the other hand, we often need to query past data. So, *at *is the normal case.

Keep in mind that there might be no value available.

### Deep dive

A `Timeline` is based on a `PhaseList`. So we can delegate most functionality provided by a timeline to a `PhaseList`. `tryApply` makes this a bit easier. Also a good show case why pipes and partial application is such a blessing.

We look backwards for the first phase with a start smaller than or equal to `at`.

We often also need to know how long this value was unchanged, so we can use the `atWithStart` function:

## Slicing a timeline

When we need to do further calculations on values in a timeline, it is often useful to slice the timeline to a specific range of interest. This speeds up further calculations by dropping unneeded data.

### Deep Dive

Different use cases needed different slicing variants, so we introduced options to specify exactly how the resulting timeline should be sliced.

Again, we delegate to `PhaseList` for the actual work:

First, we trim the start to the specified range. Then we trim the end if needed. Finally, we adjust the start according to the specified options.

## Fold over a timeline

In our systems, expenses have a state, and they pass through different phases:

To find the final state, we can fold (or loop) through all phases and calculate the state by applying the above state machine.

So, for example, if an expense was created, accepted, and then withdrawn, the overall state is *withdrawn*.

### Deep Dive

## There is way more

I think this blog post is already too long, so I’ll stop here. There are some interesting other functions in the `Timeline` module:

- `getDistinctValuesIn range`(no duplicats),- `getNonDistinctValuesIn range`(faster)
- `combine2`combines two timelines into a single timeline. Both timelines need to have the same granularity
- `choose`to choose only phases that match a predicate
- `bind`to make a single timeline out of two nested timelines (there is an inner timeline per outer phase)
- unzip to transform a `Timeline<'a * 'b, 'granularity>`into`Timeline<'a,'granularity> * Timeline<'b,'granularity>`( * means tuple in F#)

Let me know if you are interested in a deep dive into one of these.
