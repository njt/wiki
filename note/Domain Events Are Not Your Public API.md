# Domain Events Are Not Your Public API

Derek Comartin's CodeOpinion post argues that the common advice "stop publishing CRUD events, publish domain events instead" is a trap if taken literally: domain events are internal to a boundary, and publishing all of them recreates the exact coupling you were trying to escape. The fix is a deliberate integration-event model that acts as a public API contract, distinct from both your persistence model and your domain model.

---

## The argument in one paragraph

Publishing your domain events outside the boundary is equivalent to exposing your database schema through your HTTP API, because both mistakes turn private implementation details into public contracts that block evolution. The claim is falsifiable: if Comartin is right, systems that broadcast every internal domain event should show (a) consumer breakage on internal refactors that don't change business behaviour, and (b) consumers inferring business meaning from sequences of granular status events — and conversely, teams that define a separate, explicit integration-event contract should be able to change internal workflows without coordinating with consumers. The post matters because "use domain events" has become cargo-cult advice in event-driven design, and this is one of the few pieces that draws the boundary-internal/boundary-external line sharply.

---

## Key quotes

> "That's because your domain events are private. They're really no different than publishing data change events if you're exposing internal details that the rest of the system shouldn't know about."

This is the thesis stated as an equivalence claim: the CRUD-vs-domain-events distinction is a red herring; what matters is whether the event is inside or outside the contract boundary.

> "If you expose your database schema directly through your API, consumers start depending on that schema. Now when you want to change your database model, you can't easily do it because you've accidentally turned an internal implementation detail into a public contract."

The HTTP API analogy is the pedagogical core — most developers have already felt this pain with REST resources, and Comartin leverages that scar tissue to make the event case land.

> "When you make that synchronous call, you're generally asking for the state right now. You're not necessarily getting the state from when the event occurred."

The sharpest technical observation in the piece: thin ID-only events don't just add latency, they introduce a temporal correctness bug, because the consumer reacts to a past fact using present state.

> "But sometimes what looks like an event versioning problem isn't really a versioning problem at all."

Reframing versioning struggles as missing domain concepts (`TruckOrderNotUsed` vs `TruckOrderCanceledBeforeShipment`) is the most transferable idea here — it applies well beyond events.

> "Your data model is not your integration model. Your domain model is not your integration model either."

The closing rule of thumb, and the sentence most worth repeating in design reviews.

---

## Critical analysis

The non-obvious move here is the *three-model* framing. Most event-driven literature talks about two things — your data and your events — and the popular correction ("domain events, not CRUD events") only splits the world in two. Comartin splits it into three: persistence model, domain model, integration model. That third model is where all the actual design decisions live, and it's the one teams skip. The `ReasonCode: 789` example is excellent because it shows the leak happening at the field level, not the event level — you can have the "right" event name and still be coupled through its payload.

The temporal argument against thin events deserves more weight than it gets. "Fat vs thin events" is usually debated as a bandwidth/coupling trade-off; Comartin reframes it as a correctness question — the call-back consumer is answering a question about the past with data from the present. That's a genuinely different failure mode, and it's the strongest argument in the piece. It also quietly implies that the right payload is *the state relevant to the fact*, not a snapshot of the entity — which is a subtler rule than either extreme.

Where the post is weak: it's a single worked example (logistics dispatch) stretched across the whole argument, and the example is suspiciously clean. Real boundaries have contested edges — the post asserts that "not every event that occurs within a boundary should be something the rest of the system sees" but gives no method for deciding which events graduate to integration events beyond "you explicitly decide." That's true but thin; a team mid-migration gets no heuristic for the fifty events they currently publish. The overlap case (same concept, both models) is acknowledged but the guidance — "doesn't need to contain the exact same data" — raises an unaddressed maintenance cost: two schemas for one concept will drift, and someone has to keep the translation honest.

What's left out: versioning mechanics. The post says explicit contracts "give you the ability to evolve your system," but never touches compatibility strategies (versioned topics, upcasters, tolerant readers) — the hard part of actually treating events as an API. It also doesn't address event *discovery*: if integration events are a deliberate contract, who publishes the catalogue, and how do other teams find out `OrderDispatched` exists? The contract framing implies something like API documentation for events, and the post stops one step short of it.

---

## Related

- [[Modeling Facts and Reactions with Domain Events]] — Kyashif's piece covers the mechanics of raising and dispatching domain events *inside* the aggregate; this source supplies the boundary-crossing half that Kyashif's treatment stops at, sharpening his "translate to integration events at bounded-context boundaries" into a full argument that the translation is a deliberate modelling act, not a passthrough.
- [[Event-Driven vs Polling Architectures]] — Tricot's piece treats events as the trigger infrastructure for agents; this source nuances it by pointing out that event-driven design has its own coupling failure mode (leaked internal events), so "go event-driven" is necessary but not sufficient for loose coupling.
- [[The Log — Unifying Abstraction for Real-Time Data]] — Kreps' log makes everything in the stream visible and append-only; this source complicates that picture at the application layer, arguing that visibility of every internal fact to every consumer is precisely the disease — the log is the transport, but the integration contract is a separate, curated decision on top of it.
- [[The Wrong Abstraction]] — the discovery that `TruckOrderNotUsed` was really two concepts (`TruckOrderNotUsed` vs `TruckOrderCanceledBeforeShipment`) is a concrete instance of that page's thesis that the right abstraction is found by listening to how the system is actually used, not by incrementally patching the wrong one.

---
*Sources: [[raw/domain-events-are-not-your-public-api]], [[summary/domain-events-are-not-your-public-api]]*
