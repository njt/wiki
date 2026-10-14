# Change Is Inevitable — Versioning Event-Driven Systems (Bougria, NDC Oslo 2026)

Laila Bougria's NDC Oslo 2026 talk makes the case that the ignored question in event-driven design is not how to draw boundaries but how to survive changing them. Through a credit-card-service case study she walks from fat, technically-framed events to granular, business-meaningful ones, from undifferentiated payloads to public/private event discipline, from implicit wire assumptions to explicit schemas with a chosen compatibility mode, and from "we changed the contract, it's done" to a real upgrade strategy enforced by a registry.

---

## The argument, step by step

**1. Change is the blind spot.** Teams spend months on service boundaries, sync vs async, commands vs events, orchestration vs choreography — and never design for the thing that actually arrives first: change. Bougria reframes change positively (a changing system is a used system) but insists it must be prepared for up front.

**2. Fat events rot into functional coupling.** A single `CreditCardTransactionMade` with an `operationType` discriminator looks clean until reserve-funds, withdrawal costs, and currency questions arrive. Each addition teaches consumers the producer's internal rules — *functional coupling* — and removing a required field mid-life creates **poison messages** that fail deserialization on every retry.

**3. Granular, business-meaningful events.** `PaymentMade`, `RefundIssued`, `FundsBlocked`, `CostIncurred`: names you can say to a stakeholder, one meaning each, stable over time, only relevant attributes. Consumers subscribe to what they need — no switch blocks, no filters, less traffic.

**4. Public vs private events.** Private events: granular, change-prone, context-rich, safe because your team controls the consumers. Public events: self-contained, stable, minimal — "like posting a picture on the internet… you have no control over who has downloaded it." Summary events (`CardPeriodClosed`) encapsulate aggregation so the bank can move costs to yearly billing without touching any consumer. This is the **change radius** made explicit.

**5. Schemas as promises.** Beyond format coupling (JSON/Avro/protobuf) lie the unwritten assumptions: decimal precision, requiredness, units — the Mars Climate Orbiter died of a units mismatch a schema would have caught. Compatibility modes: **forward** (producers upgrade first; short-lived broker messages), **backward** (consumers upgrade first; Kafka-style retention or event stores), **full**, plus **transitive** as an attribute. Key disambiguation: schema versioning ≠ semver — deleting a field can be a non-breaking schema change.

**6. Metadata changes are always breaking.** Headers drive routing, processing, and filtering, so the CloudEvents spec standardizes the envelope (id, type, source required; data schema, content-type, time optional) and states that any metadata change warrants a brand-new event type.

**7. Upgrade strategy for real breaks.** New event type → deprecate with a removal date → dual-publish through the window → stop. Migration mechanics matter: subscribe to the new topic *before* closing the old subscription (the operations aren't atomic — reversed order loses messages at throughput), use a false filter to drain in-flight messages, deduplicate by logical id.

**8. Formalize the promise.** Compatibility is "just a promise" unless enforced. Schema registries do it but are broker-locked; she champions CNCF **xRegistry** (RC4): protocol-agnostic registry of endpoints, message definitions and schemas, emitting registry-change events so consumers learn about deprecations by subscription — restoring pub/sub's original promise of not knowing your consumers.

---

## Key quotes

> "There's always this blind spot, something that we never really think about up front… and that is change."

The talk's thesis in one line: the un-discussed topic is the one that actually breaks systems.

> "Change is not a point in time event. It is something that requires facilitation… coordination and planning."

Against the "we changed the contract, it's done" reflex — in-flight messages, delayed messages, and error-queue backlogs mean old and new contracts coexist for a window whether you plan for it or not.

> "Distributed systems are hard enough while being disciplined about sticking to the promises that we made, but they become absolute chaos if breaking our promises is easy." — quoting Clemens Vasters

The pivot to enforcement: every compatibility guarantee is a promise, so make breaking the promise mechanically hard (registry validation on egress).

> "As a consumer, I can subscribe to… version was added, schema was deprecated, message was deprecated, endpoint was deprecated, and what should I use in place of that… As a producer, I can really stop caring about who my consumers are."

The elegant close: xRegistry turns "notify your consumers" — which violates pub/sub decoupling — into just another event subscription.

---

## Key themes

- #concept **Functional coupling** — consumers absorbing producer-internal rules; the recurring villain of the talk.
- #concept **Change radius** — private events localize change; public events radiate system-wide.
- #pattern **Summary events** — aggregate inside the boundary, publish a stable, self-contained result.
- #pattern **Compatibility modes as policy** — forward/backward/full/transitive, chosen up front and published to consumers.
- #tool **CloudEvents, schema registries, xRegistry** — the standards layer that turns promises into checks.

## Analysis

This is the best practicum-level treatment of event versioning I've seen, and its strength is sequencing: Bougria refuses to let you jump to the tooling answer. Schema registries and xRegistry arrive only after a full hour establishing that most versioning pain is self-inflicted by event design — which is honest, because a registry faithfully enforcing compatibility on a badly-shaped event family just preserves the badness compatibly.

The public/private event distinction is the talk's real intellectual contribution, and the internet-photo analogy earns its keep: producers *do* lose control on publish, and treating public contracts as permanent-by-default is the correct discipline. I'd push on one tension she acknowledges but doesn't resolve: summary events move coupling rather than eliminating it — `CardPeriodClosed` consumers now depend on the producer's notion of "period," and if the semantics of the summary shift, the stable-looking contract is quietly redefined. Granularity and stability pull against each other, and "one meaning per event" is a judgment call, not a rule.

The semver-vs-schema-versioning distinction deserves wider circulation; teams sharing POCO/POJO packages with semver are systematically over-breaking because they import API discipline into data contracts.

## Related pages

This talk gives the concrete, mechanical treatment that [[Event-Driven Does Not Mean Decoupled]] complicates at the conceptual level: Bougria's functional coupling is precisely the hidden coupling that survives pub/sub, and her xRegistry closing move is a direct answer to the decoupling promise that page interrogates.

It sharpens [[Domain Events Are Not Your Public API]] from a warning into a full public/private taxonomy with compatibility modes and a deprecation protocol — the same boundary-discipline argument, engineered.

It complements [[21 Years and Counting of Eight Fallacies of Distributed Computing]]: the fallacies assumed reliable delivery and stable topology; Bougria covers the unlisted fallacy — that contracts stand still — with in-flight messages, error queues, and non-atomic subscription switches as the evidence.

It adds a data-contract dimension to [[Event Storming for Fun and Profit — Daniel Terhorst-North (Craft 2025)]]: business-meaningful event names are discovered the way event storming discovers them, but Bougria shows they must then be frozen into public contracts with a change policy.

---
*Sources: [[raw/change-is-inevitable-versioning-event-driven-systems-laila-bougria-ndc-oslo-2026]], [[summary/change-is-inevitable-versioning-event-driven-systems-laila-bougria-ndc-oslo-2026]]*
*Last updated: 2026-10-14*
