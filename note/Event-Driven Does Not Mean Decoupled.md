# Event-Driven Does Not Mean Decoupled

Derek Comartin's CodeOpinion post attacks the most common self-deception in event-driven architecture: that putting a message broker between services decouples them. His claim is that it only removes temporal coupling, and that CRUD-style change events recreate the tightest coupling of all — consumers that understand your internal data model.

---

## The argument in one paragraph

Publishing events that mirror your internal data model (`ShipmentStatusChanged`, `ShipmentAddressUpdated`) is functionally equivalent to letting other services read your database: they must interpret your state representation, and once they do, you can never change it. Swapping `ShipmentStatusChanged` for explicit business events (`ShipmentDispatched`, `ShipmentDelayed`, `ShipmentDelivered`) — and treating those public events as versioned contracts distinct from private domain events — is what actually buys decoupling. This is falsifiable in a specific way: if Comartin is wrong, then a system of services exchanging granular state-change events over a broker should be able to evolve each service's internal model freely without coordinated consumer changes. His status-`4` example says it cannot, and most teams who have lived through a schema migration under such a system will concede the point.

## Key quotes

> All you've really done is start replicating data everywhere.

The sharpest sentence in the post, and the one that reframes the whole problem: the broker hasn't removed the shared-data coupling, it has just changed the transport for it.

> There's really not much difference between publishing all these data model change events to your services and allowing those services to understand the internals of another service's database.

Comartin is careful to concede what the broker *does* buy — "You removed that temporal aspect of the coupling. But you did not remove the coupling." This honesty is what separates the post from both the event-driven evangelism and the reactive backlash.

> The consumer inferred it by understanding a combination of state changes inside another service.

The delayed-shipment inference example is the post's best concrete illustration: a consumer deducing "delayed" from location-then-date-then-other-property changes is doing implicit business logic against another service's internals, with no contract guaranteeing the inference stays true.

> Domain events are private. They're for me. They live within my local boundary. Integration events are public. They're contracts.

The inside/outside framing is the load-bearing distinction of the whole piece — and the one most teams skip, publishing whatever the aggregate raised straight onto the broker.

> Change data capture is often used in a way where event driven architecture is masking what you're actually doing, which is database replication.

The CDC passage is the most contrarian part: Comartin names a widely-recommended pattern (stream your database changes onto the bus) as the same leak of internals, just laundered through infrastructure.

## Critical analysis

What is non-obvious here is the refusal to blame the data type. The obvious reading of the status-`4` example is "use strings or enums," and Comartin pre-empts it explicitly: the problem is that the consumer must *understand your representation at all*, not how that representation is encoded. That moves the argument from a serialization nitpick to an architectural one, which is where it belongs.

The post is also honest about the legitimate case for data distribution — reporting — rather than pretending the notification pattern covers everything. But the CDC caveat is where it is weakest on practicality. "Have some type of translation" is easy to say and expensive to do: someone must now maintain a translation layer that maps every internal change to a contracted summary event, and for reporting workloads that layer is often pure overhead compared to just reading a replica. Comartin never grapples with the cost side; the post prescribes contract discipline without acknowledging that contracts have a maintenance price that grows with the number of event types.

What's left out: versioning strategy. The post says integration events need versioning and backwards compatibility "the same way you think about APIs" and then stops. How do you evolve `ShipmentDelivered` when delivery gains a partial-failure concept? Additive fields? Event envelopes? New event types? This is exactly where teams that accepted the inside/outside distinction still fail, and the post is silent on it. It also doesn't address the discovery problem — how does a consumer *find out* that `ShipmentDelayed` exists — which is the operational half of the same contract question.

Finally, the "what does the consumer need this for?" heuristic is genuinely useful but undercuts itself slightly: the local-cache smell test ("should that functionality actually live there?") is a boundary-redesign question smuggled into an event-design post. That's not wrong — it may be the deepest point in the piece — but it deserves more than two sentences.

## Related

- [[Domain Events Are Not Your Public API]] — Comartin's own earlier post makes the complementary half of this argument: domain events are internal and publishing all of them recreates coupling. This post strengthens it by supplying the positive counterpart — what integration events should look like (explicit, business-level, versioned) rather than only what they shouldn't.
- [[Modeling Facts and Reactions with Domain Events]] — Kyashif's mechanics-first treatment of raising events inside aggregates and translating to integration events at bounded-context boundaries. This source nuances it: Kyashif focuses on where the fact becomes durable, while Comartin insists the translation layer is not a formality but the entire decoupling guarantee — get the outside event wrong and the boundary work is wasted.
- [[The Valley of Webhooks]] — the polemic against webhooks-as-replication makes the same category error argument (notifications mistaken for data transfer) about a different primitive. This source complicates it: Comartin concedes data distribution is sometimes legitimate (reporting) provided it goes through a contracted translation, a more permissive position than the webhook polemic's flat rejection.
- [[Event-Driven vs Polling Architectures]] — Tricot's piece treats the trigger mechanism as load-bearing infrastructure. This source nuances it by shifting the question one level up: choosing events over polling settles temporal coupling only, and the harder design decision is what the events mean.

---
*Sources: [[raw/event-driven-architecture-and-coupling-youre-not-as-decoupled-as-you-think]], [[summary/event-driven-architecture-and-coupling-youre-not-as-decoupled-as-you-think]]*
