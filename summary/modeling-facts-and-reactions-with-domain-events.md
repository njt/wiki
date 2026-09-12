---
url: https://deniskyashif.com/2026/07/25/modeling-facts-and-reactions-with-domain-events/
title: "Modeling Facts and Reactions with Domain Events"
author: Denis Kyashif
date_fetched: 2026-09-13
date_published: 2026-07-25
topics:
  - software-engineering-craft
  - distributed-systems
---

# Modeling Facts and Reactions with Domain Events

Part of Denis Kyashif's ongoing series on Domain-Driven Design, this entry covers **domain events**: how to record what happened in the domain and let each consequence react to it independently, without the fact having to know about any of its reactions.

## The fact/consequence split

Kyashif starts from the shape most business logic eventually takes — "when something happens, something else should happen" — and shows how the naive transaction script (`placeOrder`; save; send email; notify fulfillment; update analytics) couples a domain fact to its consequences. The fix is to ask "what happened, and who needs to know?" — record the fact (`OrderPlaced`: past tense, immutable, raised where the invariants are enforced) in one place, and let each consequence own its own handler, failure mode, and retry behavior.

## Mechanics

Events are raised inside the aggregate that made the decision, not wherever a row happens to be inserted, so every path through the model records the same fact. The unit of work collects events and dispatches them to handlers only after the transaction commits; each handler reads as "when this happened, do that." Cross-aggregate reactions (reserve inventory on `OrderPlaced`) run as a second transaction, making the gap an explicit, eventually consistent design decision rather than a hidden coupling — one aggregate per transaction, and a need to span two is a design smell. Crossing a bounded context requires an **integration event** (`OrderReadyForFulfillment`) that the consuming context translates into its own local commands; when contexts are separate processes, asynchronous delivery (Kafka, RabbitMQ, Azure Service Bus) plus a transactional outbox is the reliable path.

## Restraint

The piece closes with a caution against event maximalism: not every state change deserves an event, and an event per setter produces noise rather than a model. Prefer a direct call when the dependency is one clear operation, and choose synchronous vs. durable asynchronous delivery only after the transaction, consistency, and failure requirements are clear.
