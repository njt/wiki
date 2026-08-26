# Vertical Slice Architecture

JasperFx Software's Wolverine guide to organizing code by feature or use case rather than by horizontal technical layer, using the "A-Frame Architecture" to isolate business logic from infrastructure without Clean/Onion Architecture's layering ceremony. It's a concrete, opinionated alternative to the MediatR-plus-Minimal-API VSA that dominates .NET.

---

## Key Quotes

> "Generally try to organize code by feature or use case rather than by horizontal technical layering."

The one-line definition of VSA, and the frame the whole article hangs on. The contrast isn't just aesthetic: layering forces you to reason about an *entire* data-access or domain layer at once, while slicing by feature matches how you actually work — one system input at a time.

> "Effective test coverage is paramount for sustainable development. More than layering schemes or the right abstractions or code structure."

The article's most important claim, and its ordering is the point. Clean Architecture treats the layer diagram as the load-bearing structure; the Wolverine team treats *the test suite* as load-bearing and everything else as negotiable. Layering is a means to testability, not an end — so if you can get testability a cheaper way, drop the layers.

> "We are recommending that you largely forgo wrapping any kind of repository abstractions around your persistence tooling, but instead, purposely seek to shrink down the call stack depth."

The most controversial line, aimed squarely at Clean Architecture's repository interfaces. Their value judgment is explicit: simpler code matters more than the option to swap persistence tooling later. The A-Frame's isolation — business logic in one place, infrastructure in another, coordination on top — is what lets a handler call `IDocumentSession` directly without the business logic becoming coupled to Marten.

> "Look Ma, no mocks anywhere in sight!"

The payoff of the pure-function approach. By returning `Storage.Insert(order)` instead of calling the session, the handler becomes testable by walking up and asserting on inputs and outputs — no mocks, no extra projects, no database. Testability *and* simplicity, which they insist aren't in tension once you stop treating layering as the price of testability.

> "An idiom with Wolverine development is to largely utilize `Validate` methods to make the main handler or endpoint method be for the 'Happy Path'."

The `Validate()` compound handler returns `ProblemDetails` to short-circuit invalid input, leaving the main method as a clean happy path. They call it Railway Programming without the ceremony — and the honest framing (built-in, low-ceremony) beats pretending the concept is new.

---

## Key Themes

- #pattern — Vertical Slice Architecture: organize code by feature, not layer
- #pattern — A-Frame Architecture: business logic / infrastructure / coordination as the three responsibilities
- #tool — Wolverine: side effects, cascading messages, compound handlers, Wolverine.HTTP endpoints
- #tool — Marten: document persistence and event sourcing as the "Critter Stack" half (with Wolverine)
- #concept — Pure-function handlers as the route to mock-free testability
- #pattern — Transactional outbox (durable local queues + inbox) for resilience without ceremony

---

## Critical Analysis

**The real argument is about what's load-bearing.** Strip away the Wolverine specifics and this is a debate about where architecture earns its keep. Clean/Onion Architecture puts its faith in layering and abstractions to preserve the *option* of change (swap the database, swap the framework). VSA puts its faith in test coverage and locality to preserve the *ability* to change. The Wolverine team's bet — that a fast, reliable test suite does more for sustainable development than any layer diagram — is well-argued, and their point that excessive layering hides database misuse is a real, under-discussed cost: it's hard to see how the system uses the DB when the query is buried three abstractions down.

**But the pitch has a built-in conflict of interest.** This is Wolverine's own documentation selling Wolverine. The "you could use us as a MediatR replacement, but you'll get better results leaning into our special sauce" framing is genuinely useful — side effects and cascading messages *do* remove ceremony MediatR can't — but it also conveniently routes you away from the portable pattern toward the proprietary one. The A-Frame idea is framework-agnostic; the pure-function handlers only work because Wolverine ships `Insert<T>`, `Delete<T>`, and transactional middleware. That's not a knock, but it's worth naming: the testability win is purchased by adopting a framework, which is itself the kind of coupling the article elsewhere shrugs off.

**The query-side dodge is honest but telling.** "We admittedly don't have nearly as much to say about the query side" is refreshing candor, but it's also the hole in the whole VSA/CQRS story. The write side gets elegant pure functions and side-effect declarations; the read side gets "just call the persistence tool directly in a GET endpoint and rely on integration tests." That asymmetry is fine for simple reads, but the moment queries need reshaping, composition, or cross-slice aggregation, the article's "put the query in the same file" advice stops scaling. Compare [[CQRS Pattern in C# and Clean Architecture]], which has the inverse gap — all Commands, no Queries — and you start to see that the read side is where everyone's tutorial runs out of runway.

**Where it lands on Locality of Behaviour.** The "put data access germane to one slice in that slice's file" recommendation is [[Locality of Behaviour]] wearing a .NET suit: behaviour should be obvious on inspection of the unit that contains it. The A-Frame is the mechanism that lets you colocate the query with the handler *without* coupling business logic to infrastructure — same tradeoff Gross names between LoB and SoC, resolved at the feature boundary rather than the DOM element.

**The best idea is the shrink-the-stack heuristic.** "Shrink the call-stack depth" — handler → service A → service B → repository C → persistence D — is a diagnostic worth keeping regardless of framework. Every hop is a place to lose context, and the article's insistence that you count hops as a cost is more durable than any of the Wolverine-specific machinery around it.

---

*Sources: [[raw/vertical-slice-architecture-html]], [[summary/vertical-slice-architecture-html]]*
*Last updated: 2026-08-26*
