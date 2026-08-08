# HTTP API Design Guide (Heroku)

The ur-text of modern HTTP+JSON API design. Extracted from Heroku's Platform API work circa 2013 by Wesley Beary (geemus), this guide crystallised a set of conventions that became de facto industry standards: API versioning via the `Accept` header, structured error responses with machine-readable IDs, UUIDs as resource identifiers, cursor pagination, and the flat-path-over-nested-routes rule. Short enough to read in ten minutes, comprehensive enough to settle any argument.

---

## Key Quotes

> "We are aiming for consistency and focusing on business logic while avoiding the bikeshedding of API design."

The mission statement. This guide exists because the Heroku team kept having the same arguments on every new API, so they wrote down the answers and got back to work. The writing style is declarative — no justifications, no alternatives considered, just rulings.

> "Do not accept only names to the exclusion of IDs."

On non-ID dereferencing: human-friendly names are a convenience, not a replacement for stable identifiers. Names change; UUIDs don't. This rule prevents calling something by name forever and getting trapped when it inevitably gets renamed.

> "Redirects allow sloppy/bad client behaviour without providing any clear gain."

On TLS: if you redirect HTTP to HTTPS, the sensitive data in the initial unencrypted request has already leaked. The redirect pattern trains clients to be careless. Better to refuse the connection entirely.

> "Do not make backwards incompatible changes within that API version. If backwards-incompatible changes are needed, you should create a new API with an incremented version number."

The immutability contract. Once published, an API version is a promise. This is the same principle Sean Goedecke articulates in [[Good API Design]] — "WE DO NOT BREAK USERSPACE."

> "Return an empty array rather than null when there are no entries."

A small rule with outsized consequences. Every API consumer that crashes on `null` vs `[]` is a bug the API designer created, not the consumer. Consistent types are worth more than expressive nullability.

## Key Themes

#api-design #software-craft #pattern #tool

**Convention over configuration.** The guide doesn't explain *why* each rule exists — it just states the rule. This is API design as infrastructure: the fewer decisions you have to make, the more attention you have for the business logic that actually differentiates your product. The guide *is* the bikeshed-avoider.

**Headers over query params.** Versioning goes in `Accept` headers, not URL prefixes. Metadata belongs in headers, not paths. The path identifies the *what*; headers communicate the *how*. This is the cleanest expression of HTTP's separation-of-concerns design, and most APIs that break this rule end up regretting it.

**Minimum viable specification.** At roughly 20 short rules, this guide is deliberately minimal. Compare to JSON:API (dozens of pages on compound documents alone) or OpenAPI (a full specification language). The Heroku guide bets that consistency matters more than completeness. You can add details later; you can't remove complexity.

**The flat-path rule.** `/apps/{app_id}/dynos` not `/orgs/{org_id}/apps/{app_id}/dynos/{dyno_id}`. Nesting communicates hierarchy; every level of nesting makes the API harder to version, harder to route, and harder to document. Root-accessible resources with scoped collections is the right trade-off.

**Structured errors are a contract.** The guide's error format — machine-readable `id`, human-readable `message`, optional resolution `url` — predates RFC 7807 (Problem Details) by years. The insight is that errors are part of the API surface, not a failure mode. Clients should be able to branch on error types programmatically.

**Machine-readable schema as a first-class artifact.** The guide recommends prmd (JSON Schema + Markdown generation) as a unified source of truth that produces both machine-readable validation and human-readable docs. This "schema-first" approach predates OpenAPI/Swagger and in many ways remains cleaner — one tool that does one thing.

## Critical Analysis

This guide is a fossil that didn't fossilise. Written in 2013, it reads like it was written yesterday. Nearly every recommendation has aged well: the TLS absolutism (Snowden was the same year), the `Accept`-header versioning (still the most elegant approach, still barely adopted), the structured error format (now standardised as RFC 7807), the UUID preference (auto-increment IDs are a Postgres habit that leaks into APIs and shouldn't).

The guide's brevity is both its greatest strength and its most conspicuous omission. There's no discussion of pagination beyond the Range-header mention. No treatment of filtering, sorting, searching, or sparse fieldsets. No rate-limiting strategy beyond "use token buckets and report remaining tokens." No mention of idempotency keys, which Heroku itself would later champion. These aren't bugs — the guide is a starting point, not an encyclopedia — but anyone using it as their sole reference will hit these gaps within the first month of production.

The `application/vnd.heroku+json; version=3` pattern is elegant and thoroughly ignored by the industry. Most teams stick versions in the URL (`/v3/...`) because it's visible and debuggable, even though the guide is correct that the `Accept` header is architecturally purer. The guide's other recommendations won; this one lost.

The Heroku context matters. This guide was written by a platform team that owned the entire API lifecycle — they could enforce these rules with tooling and code review. A team inheriting an existing API or building a public API for the first time will find the guide aspirational rather than actionable. That's not a criticism of the guide; it's a reminder that good API design requires organisational commitment, not just a style guide.

Compared to [[Good API Design]], the Heroku guide is the prescriptive, enumerable companion. Goedecke gives you the philosophy (boring is a feature, product trumps interface); the Heroku guide gives you the checklist (downcase paths, UUIDs, structured errors, minified JSON). You need both: the philosophy tells you *why* to design boring APIs; the checklist tells you *how*.

What's remarkable in retrospect is how many things that are now obvious were not obvious in 2013. Structured errors weren't standard. JSON:API didn't exist. GraphQL didn't exist. REST was still being debated as a concept. This guide was quietly being right about everything while the industry fought format holy wars. The winning strategy wasn't being clever; it was being consistent and boring — exactly what the guide recommends.

**A note on terminology: this guide describes JSON APIs, not RESTful systems.** Per [[Components of a Hypermedia System]], Fielding's original definition of REST requires hypermedia controls and self-describing messages — properties that a JSON API without hypermedia links structurally cannot satisfy. The Heroku guide's conventions are excellent API design, and they remain the industry's de facto standard for HTTP+JSON services. But in Fielding's taxonomy they describe a data API, not a RESTful hypermedia system. The distinction matters because hypermedia APIs don't need versioning (the response encodes available operations) while JSON APIs do (the client must know URL structures and methods from documentation). The Heroku guide's versioning rules are necessary precisely because JSON APIs aren't RESTful.

## Cross-links

- [[Good API Design]] — the philosophical companion: Goedecke explains *why* boring APIs win; this guide spells out *how* to build one
- [[Software Engineering Craft]] — hub page covering API design, error handling, and reliability as enduring fundamentals
- [[Designing a Passively Safe API]] — Albaugh's mechanical engineering approach to API safety: idempotency lifecycle, recovery points, and transient error classification
- [[Your Backend Is Full of Hidden Workflows]] — the flat-path rule in this guide fights the coordination complexity that accretes in deeply nested resource models
- [[Smart Models Dumb Pipes]] — the guide's "separate concerns" rule is the same end-to-end principle: paths identify, bodies carry, headers communicate
- [[Better Error Messages]] — the guide's structured error format anticipates the principle that errors are part of the API surface
- [[Queues Don't Fix Overload]] — token bucket rate limiting in the guide aligns with Fred Hebert's "identify the bottleneck, then back-pressure"
- [[Stevey's Google Platforms Rant]] — the strategic argument that makes guides like this existentially important: Yegge's case that every service interface must be "designed from the ground up to be externalizable," and that product companies that skip this step get replaced by platform-ized competitors

---
*Sources: [[raw/http-api-design-guide]]*
*Last updated: 2026-07-08*
