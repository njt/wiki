# Amundsen's Maxim

A one-page site dedicated to Mike Amundsen's 2012 API-design maxim: "your data model is not your object model is not your resource model is not your message model." The page explains each of the four separations, traces the maxim's lineage to Fielding's REST constraints and Evans's Bounded Contexts, and collects citations showing it circulating in real API-design discussions.

---

The maxim is a warning against coupling: if the API mirrors the database schema, schema changes break clients; if resources mirror objects, you get rigid CRUD-shaped APIs designed around internals rather than use cases; if messages mirror resources, you ship payloads bloated beyond what any client needs. Each layer — storage, code, resource, wire format — should be designed for its own purpose.

The site grounds the principle in two traditions: Fielding's REST constraints (abstraction between client and server) and Eric Evans's DDD (different bounded contexts demand different models). It also lists real-world citations — CodeOpinion on OData/EF, Nordic APIs on REST state machines, APIscene (2024), StackOverflow threads — suggesting the maxim has become a standard shorthand in API circles.

---

## Key Quotes

> "Remember, when designing your Web API, your data model is not your object model is not your resource model is not your message model."

The chain structure is what makes it memorable: it's not "don't couple two things," it's a four-way refusal, which names the specific failure mode of every lazy API — the entity-to-table-to-JSON pipeline where one C# class wears four hats.

> "Mapping objects directly to resources often leads to rigid, CRUD-based APIs that don't evolve well. Resources should be designed around use cases, not internal object structures."

This is the strongest of the three separations and the one most often violated, especially by frameworks that make "expose your ORM entities over HTTP" a one-command affair.

## Key Themes

#concept #pattern

## Analysis

The maxim is correct and, precisely because it is correct and quotable, it functions less as advice than as a diagnostic of tooling defaults. Every framework that promises "your database, but as an API" — OData over Entity Framework, auto-generated REST from schemas — is a commercial product built on breaking the maxim, because collapsing the four models into one artifact is where the productivity comes from. The page itself, a hagiographic single-purpose site for a maxim, is candid about this being Amundsen's personal brand (he's cited by alliances, conferences, and his own books), so read it as advocacy rather than neutral reference. It also predates the agentic era, and its relevance there is untested: when the client is a coding agent reading your API, the temptation to collapse resource and message models — just give the agent the whole schema — is stronger than ever, and whether the separation still pays off for machine clients is an open question.

## Related Pages

This source sharpens [[Good API Design]] by giving that page's general principles a specific, nameable rule about the four models. It complicates [[Object-Relational Impedance Mismatch]] — the object/data separation is the ORM problem restated at the API boundary, and the maxim explains why ORMs alone can't fix it. It nuances [[Domain Events Are Not Your Public API]] with the message/resource separation: events are a message model, and exposing them as resources conflates two layers. And it connects to [[HATEOAS — Hypermedia as the Engine of Application State]], since the resource-vs-affordance framing Amundsen advocates is exactly what hypermedia designs require.

---
*Sources: [[raw/amundsens-maxim-com]], [[summary/amundsens-maxim-com]]*
*Last updated: 2026-10-26*
