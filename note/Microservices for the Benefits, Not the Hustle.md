# Microservices for the Benefits, Not the Hustle

Oliver Wolf's sharp, practitioner-grounded defense of microservices that explicitly de-prioritizes scalability -- the reason everyone reaches for microservices -- in favor of changeability, the reason they should. Written in 2023 but more relevant as agent-generated code makes architectural boundaries both cheaper to build and more essential to get right.

---

## Key Quotes

> "Making a system scalable -- or cheaper to scale -- is the first benefit that comes to mind when you think about microservices."

Wolf opens by naming the elephant. Most teams cite scalability as their microservices motivation, but for the vast majority of systems, the load-based break-even is nowhere near. This is the article's most useful move: it clears away the false reason so the real ones can breathe.

> "different responsibilities are placed in different services"

The single responsibility principle at architectural scale. Not novel, but Wolf's treatment of *why* this works -- hard repo boundaries that resist deadline pressure -- is more honest than most. Developers don't ignore subsystem boundaries because they're undisciplined; they ignore them because the monolith makes boundary violations cheap in the moment and expensive only later. Microservices invert that calculus.

> "choreography should be preferred over orchestration"

The strongest opinion in the piece and the one most likely to be wrong in practice. Choreography (events on a bus, services react autonomously) reduces coupling at the cost of observability -- when something breaks, nobody owns the transaction. Wolf acknowledges REST for CRUD but doesn't reckon with the debugging hell of pure event-driven architectures. The pragmatist's position: start with orchestration, move to choreography only when the coupling costs become measurable.

> "each service must have its own logical database schema"

The rule that separates microservices from a distributed monolith. Shared databases are the cheat code that makes microservices feel easy initially and impossible later. Wolf is right to be absolutist here.

> "The service can be rewritten and redeployed in 2 weeks." -- Jon Eaves

A sizing heuristic that's more useful than lines-of-code or team-size because it measures *comprehension velocity*, not static properties. If you can't rewrite it in two weeks, you don't understand it well enough to maintain it.

## Key Themes

#microservices #architecture #SOA #choreography #software-craft #scalability

- **Changeability over scalability.** The thesis. Microservices reduce the cost of change by enforcing boundaries that monoliths allow to erode. This is the argument that should be made in every architecture review.
- **Boundary enforcement as the mechanism.** Not communication patterns, not deployment independence, not team topology -- those are downstream effects. The mechanism is: separate repos make wrong-code-in-wrong-place physically harder.
- **Sizing is the hard problem.** Wolf's reframe -- "which responsibilities together?" rather than "how big?" -- is genuinely useful and under-discussed in microservices literature.
- **SOA continuity.** Wolf is explicit that microservices ARE SOA, just smaller and with lighter protocols. The SOA->microservices transition wasn't a paradigm shift; it was the same ideas with better tooling and less XML.

## Critical Analysis

This is a good article that could have been great with more skepticism. Wolf is a believer, and the piece reads as an advocacy document. The three purposes (changeability, reuse, operations efficiency) are well-argued, but the risks section is underweight relative to the benefits section -- two short risks against three detailed purposes, and neither risk gets the practical treatment the benefits do.

**What's right:** The reframing of microservices around changeability rather than scalability is correct and important. The decomposition dimensions (actions on entities, aspects of an action) are concrete where most architecture advice is hand-wavy. The "nanoservices" warning -- that too-small services create their own problems -- shows Wolf isn't a zealot.

**What's missing:** Wolf treats choreography vs. orchestration as a settled question when it's one of the most contested design decisions in distributed systems. Event-driven architectures create debugging, testing, and observability challenges that the article doesn't acknowledge. The "two week rewrite" sizing heuristic is elegant but assumes the team that built the service still exists and remembers it -- what about the service nobody has touched in 18 months?

**What's dated:** The article predates the coding agent explosion. In an agent-heavy development world, microservice boundaries become even more valuable -- they limit the blast radius of agent-generated code -- but they also become easier to create, making the nanoservices risk more acute. Wolf's "hard boundaries" argument gets stronger, but his sizing advice needs updating for a world where spinning up a new service is a prompt, not a project.

**The unconvincing part:** Purpose 3 (operations efficiency through selective scaling) is the weakest section. For most systems, the operational complexity of N services outweighs the hardware savings of selective scaling. Wolf knows this -- he leads with it -- but includes it anyway, which muddies the argument.

## Cross-Links

- [[Distributed Systems]] -- the observability gap Wolf's choreography preference creates; microservice architectures need Jaeger/OpenTelemetry tooling that agent orchestration systems also lack
- [[Aspire]] -- the tooling that closes that gap and lowers the cost of Wolf's changeability thesis: a code-first app model that injects OTel wiring and runs the whole topology locally via its Developer Control Plane, so the boundary is one builder call away
- [[Software Engineering Craft]] -- this is software architecture craft: knowing when boundaries pay for themselves
- [[Make the Easy Change Hard]] -- Wolf's argument inverted: microservices make easy changes hard (network calls, distributed transactions) so that hard changes become tractable (independent deployability, bounded contexts)
- [[Building an AI Agent in Rails (Ionescu)]] -- a monolith that works; the counterpoint to Wolf's thesis
- [[Systems Ideas That Sound Good]] -- microservices applied without Wolf's discipline are exactly the kind of pattern that "fails 9 out of 10 times"
- [[Simplicity in the Age of AI-Assisted]] -- when agents can generate microservices on demand, Wolf's sizing question becomes the only question

---
*Sources: [[summary/microservices-for-the-benefits-not-the-hustle]]*
*Last updated: 2026-05-15*
