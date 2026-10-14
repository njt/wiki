---
url: https://www.planetgeek.ch/2026/10/12/event-sourcing-a-close-look-at-the-projector-for-values/
date_fetched: 2026-10-14
---

In this part of my event sourcing series, we look at implementing the projector for values (uni-temporal events / normal event sourcing). You’ll see the full source code for our ValueProjector, which projects events into a single value representing the latest state.

This brings together many of the things you’ve seen in earlier posts in this series.

## Projection and some helper types

First, we need some types that are used by the `ValueProjector`.

A `Projection` states whether a value has never existed (e.g. when querying data that does not exist at all), is existing, or was deleted (is bygone). In the case of the deleted data, we still know the value it had before it was deleted.

The type alias `projection<'a>` is for fancy type definitions. Instead of `Projection<Employee>`, we can write `Employee projection`.

`EventsForProjection(s)` are used to give a list of events more meaning. Our functions that query the database return these types.

The `Streamlining` types are used to allow accept, reject workflows as seen in an earlier post.

All of the following code snippets are from the same file. I split them up to make it easier to explain (and making screenshots).

## Projection Configuration

As you have seen through this blog post series there are different ways to project events. For all these kinds there exists a discriminated union that is used to specify how each event has to be interpreted:

The `ProjectionActionConfiguration` is used to select the kind of the projection. The `ProjectionConfiguration` is the type that has to be passed to the `ValueProjector` (see below).

## Sorting events

Before the events can be projected, they need to be sorted. There exist different sort functions for the different kinds of projections:

Create, update, delete events are sorted first by their application and second by their event type. For example, a create and update event with the same application timestamp are always sorted create first, update second.

Events that are streamlined first, are sorted the same way.

The final special case is events with an explicite update order. This is for cases where we anticipate multiple update events with the same application and it is important to have a deterministic order in which these updates are executed. A rare case, but still a case we have in our domain.

## Projecting create, update, and delete events

A simple `List.fold` is used to project these events:

The fold function represents a state machine that looks at the current event (whether it represents a create, update, or delete action) and the current projection, and acts accordingly. Invalid events are ignored (e.g. an update after a delete event).

## Projecting set and remove events

Set and remove projection is similar with the difference that multiple set events are accepted:

## Projecting ordered updates events

We map them so that we can use the projection function for `CreateProjectionAction` for `OrderedUpdatesProjectionAction`.

## Projecting streamlined events

Streamlinging means that we get rid of rejected events as we have seen in Event Sourcing: two ways to enable accept/reject workflows. So we first loop through all accept and reject events to see what triggers are considered accepted, rejected, or still undecided. We keep the data in a dictionary. I use a mutuable dictionary because it is faster than building a map through folding over the events. Then, we loop through the events again, decide whether we need to keep the event – depending on its acceptance state.

Finally, we fold over the filtered events to get the projection – the same way we did for the normal create/update/delete events.

## Projection Functions

Depending on the configuration that is passed to the projection function, we sort and project the passed events:

Finally, the functions that are called from the outside providing the different kinds of projection needed:

- project a list of events for a single projection
- project with an initial projection (apply further events)
- project multiple projections (e.g. multiple employees): need a group function that returns the unique identifier to identify each projection
- project multiple projections and return them in a dictionary with the key being the group identifier
