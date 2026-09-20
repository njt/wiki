---
url: https://codeopinion.com/event-driven-architecture-and-coupling-youre-not-as-decoupled-as-you-think/
title: "Event-Driven Architecture and Coupling — You're Not as Decoupled as You Think"
author: Derek Comartin
date_fetched: 2026-09-20
topics:
  - distributed-systems
  - software-engineering-craft
---

Derek Comartin argues that adopting event-driven architecture and a message broker removes only the *temporal* coupling between services, not the coupling itself. If you publish CRUD-style events (`ShipmentStatusChanged`, `ShipmentAddressUpdated`) and every consumer maintains a local copy of your data, you have merely replaced a shared database with message-based data replication — consumers still understand your internals, and you can no longer change your state model without breaking them.

The fix is to publish explicit, business-meaningful integration events (`ShipmentDispatched`, `ShipmentDelayed`, `ShipmentDelivered`) so consumers never have to infer what happened by reverse-engineering combinations of state changes. Comartin draws a sharp line between domain events (internal, mutable, granular, owned within a service boundary) and integration events (public contracts that need versioning and backwards compatibility, like an HTTP API). He concedes a legitimate use for data-distribution events — reporting — but insists CDC pipelines must translate internal database changes into contracted summary events rather than leaking schema directly.

He closes with a design heuristic: ask what the consumer actually needs the event *for*. Reacting to something that happened is a good answer; caching a copy of the data is a smell that your service boundaries are misaligned. The title thesis: event-driven does not mean decoupled — what matters is what the events mean, not that a broker exists.
