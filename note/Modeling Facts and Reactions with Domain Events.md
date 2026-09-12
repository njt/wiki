# Modeling Facts and Reactions with Domain Events

Denis Kyashif's DDD-series entry on domain events: separate the **fact** (an order was placed) from its **consequences** (confirmation email, fulfillment, analytics) by raising a past-tense, immutable event inside the aggregate where the decision is made, dispatching to independent handlers after commit, and translating to integration events at bounded-context boundaries. It is a mechanics-first treatment — where the event is raised, who dispatches it, what the payload carries, and when the fact becomes durable — that most event-driven writing skips on its way to the broker. #concept #pattern

---

## Key Quotes

> "Most business logic eventually boils down to the following shape: When something happens, something else should happen."

The whole article in two sentences. The naive transaction script glues the halves together; the entire domain-events pattern is about giving each half its own owner — the model owns *what happened*, handlers own *what happens because of it*.

> "The important domain fact is that an order was placed. Sending a confirmation email is a consequence of that fact, not part of the fact itself."

This is the load-bearing distinction. Once you accept it, everything else follows: naming (past-tense facts, not imperative tasks), ownership (the aggregate records, handlers react), failure handling (each consequence retries independently instead of poisoning the order write), and the observation that a failure reported as "the order failed" invites duplicate orders while a false success strands follow-up work.

> "Events are named in the past tense because they are historical records. A handler cannot reject `OrderPlaced`; it can only decide how to react."

Naming is doing semantic work here. `PlaceOrder` is a command — an intention that can be refused. `OrderPlaced` is a fact — it already happened, and the only question left is what to do about it. The refusability test is the cleanest discriminator between the two that I've seen in a practitioner piece.

> "`OrderPlaced` does not mean merely that a row was inserted. It means the model accepted the request and satisfied its invariants, so the event belongs to that decision."

The placement rule, and the article's most portable insight. Raising the event *inside* `Order.place` — rather than in the application service or the controller — means checkout, the admin tool, and the data import all record the same fact without anyone remembering to add the follow-up list at each call site. That "we have to remember to include the same set of subsequent actions at every call site" failure mode is exactly how hidden workflows accrete.

> "This is a second transaction, not an extension of the first. For a brief period the order exists but stock is not yet reserved: the two aggregates are eventually consistent."

The honest trade. Rather than papering over the gap with one big transaction, the design makes the gap explicit — one aggregate per transaction, eventual consistency across them, and "having to update two aggregates within a single transaction might indicate a design problem." The payoff is availability: orders keep being accepted while inventory resolves shortages.

## Themes

- **#concept Facts vs. consequences** — the core split; a domain event records what happened, and no handler can reject it.
- **#concept Command vs. event** — intent (refusable, imperative) vs. outcome (immutable, past tense).
- **#pattern Raise at the decision point** — events are constructed only by the aggregate that enforced the invariants; the unit of work collects and dispatches after commit.
- **#pattern One aggregate per transaction** — cross-aggregate work is a second transaction; the consistency gap is modeled, not hidden. The exception ("may sometimes be justified") is acknowledged but fenced.
- **#pattern Integration events** — a context's domain language doesn't cross its boundary; consumers translate `OrderReadyForFulfillment` into local commands rather than consuming another context's aggregates.
- **#concept Restraint** — event-per-setter is noise; "events should expose meaningful facts within our business domain and not hide dependencies."

## Critical Analysis

**What lands.** The sequencing is the strength: problem → naming → raise point → dispatch → aggregate boundaries → context boundaries → restraint. Each step answers the question the previous one opens, and the pseudocode is language-agnostic enough to port anywhere. The payload guidance ("prefer lean events, unless you have a clear reason not to" — identifiers and values, never the aggregate's internal representation) is the kind of small judgment call that most pattern write-ups omit.

**What's deferred.** The article is admirably honest that its dispatcher is a toy: synchronous, in-memory, non-durable, with "error handling and retries omitted for brevity" appearing twice. Those omissions are precisely where event-driven systems live or die. The transactional outbox is name-checked twice — "look into the transactional outbox pattern", "reliable publication usually requires a transactional outbox or an equivalent mechanism" — but never explained, which is a gap when the article otherwise builds every concept from first principles. Ordering, event-contract versioning, and deduplication get the same wave: "handlers should still be safe to repeat" is the right instinct with no mechanics behind it.

**The restraint section is the most valuable part.** Event maximalism is a real failure mode — teams discover the pattern, then emit an event for every setter and drown their own observability in noise. "Not every state change deserves an event" and "don't hide dependencies that would be easier to understand if they were explicit" give permission to *not* use the tool, which is what separates a pattern introduction from a pattern advertisement.

**Where it sits in the DDD revival.** Read alongside the arguments that DDD matters more when agents write the code, this is the supply side of that claim: the concrete mechanics — raise the fact at the decision, translate at the boundary — that make a model an agent can't accidentally flatten. `OrderPlaced` vs. `SendConfirmationEmail` is the ubiquitous-language discipline rendered as a naming decision a model can actually obey.

## Related Pages

- [[Observer Pattern to Event-Driven Architecture in Dart]] — The same decoupling thesis ("separating what happened from what should happen because of it") told from the client-app side. This source strengthens it with the server-side rigor that journey lacks: the aggregate as the raise point, the unit of work as the dispatcher, and payload discipline for what a fact carries.
- [[DDD Matters More When AI Writes Your Code]] — Smółka argued DDD's modeling discipline matters more when agents write code, but stopped before explaining how a team actually encodes aggregate and context boundaries. This source supplies exactly that missing mechanics — raise events where invariants are enforced, translate to integration events at the boundary — and demonstrates the precise-naming discipline (`OrderPlaced`, not `SendConfirmationEmail`) that Smółka claimed agents need.
- [[State-Oriented Consistency]] — That page argues consistency is a per-piece-of-state question, not a system-wide default. This source reaches the same conclusion from the DDD side — the consistency boundary is the aggregate, one per transaction — and nuances it by insisting the resulting gap be modeled explicitly as an event chain rather than absorbed silently.
- [[Event-Driven vs Polling Architectures]] — Tricot's page owns the delivery half of event-driven design: contracts, retries, idempotency, and why webhooks alone are a production trap. This source owns the modeling half — what an event even *is* before you choose how to ship it — and its closing sequencing (model the fact first, choose delivery only after consistency and failure requirements are clear) is the bridge between the two.

---
*Sources: [[raw/modeling-facts-and-reactions-with-domain-events]], [[summary/modeling-facts-and-reactions-with-domain-events]]*
*Last updated: 2026-09-13*
