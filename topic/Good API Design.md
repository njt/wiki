# Good API Design

Sean Goedecke's practitioner's guide to API design, distilled from building REST, GraphQL, and CLI APIs both public and private. The core thesis: good APIs are boring on purpose, because any cognitive load the consumer spends on the interface is stolen from the problem they're actually solving. The hard part isn't making the API clever -- it's making it unchangeable enough to trust and simple enough to adopt.

---

## Key Quotes

> "Any time they spend thinking about the API instead of about that goal is time wasted."

> "WE DO NOT BREAK USERSPACE" -- invoking Linus Torvalds' principle for downstream stability.

> Versioning is a "last resort" -- simultaneously serving old and new versions creates maintenance nightmares.

> Facebook and Jira maintain poor APIs but command usage through market dominance. "Flawless API design cannot salvage unwanted products."

## Key Themes

#api-design #software-craft #simplicity

**Boring is a feature.** Familiarity over novelty. If a developer has to read your docs to understand the basics, you've already lost.

**Immutability as a design constraint.** Once published, an API is a promise. Additive changes only. Versioning exists but is expensive -- Goedecke puts it alongside "things you do because you have to, not because you want to."

**Product > interface.** The API's success is determined by the product behind it, not the interface in front of it. Poor architecture produces poor APIs because interfaces reflect resource structures.

**Accessibility over security maximalism.** Support long-lived API keys. Many consumers are salespeople, students, hobbyists -- not engineers. OAuth is the right answer for the wrong audience.

**Idempotency where it matters.** Payment-class operations need idempotency keys. Read operations don't. DELETE is naturally idempotent. The pragmatic middle: make it available, don't make it mandatory.

**Operational defense.** Rate limits are not optional. The Zendesk anecdote -- third-party developers building an unintended chat system that overwhelmed backend servers -- is a perfect illustration of why APIs need killswitches.

**Cursor pagination, always.** Offset-based pagination degrades at scale because databases must count through every offset. Cursor-based (`WHERE id > cursor`) stays constant. Accept the learning curve now or pay the migration cost later.

**GraphQL skepticism.** Three strikes: high barrier to entry for non-engineers, query arbitrariness complicating caching, fiddly backend implementation. Use it only when you must.

**Internal APIs are different.** Your colleagues are engineers. You can break things, use complex auth, move fast. But idempotency for critical operations and operational safeguards still apply.

## Critical Analysis

This is the kind of article that's more useful for what it excludes than what it includes. Goedecke explicitly skips REST-vs-SOAP, JSON-vs-XML, and OpenAPI-vs-Markdown debates -- the format holy wars that consume most API design discussions. What remains is the structural skeleton: immutability, idempotency, pagination, rate limiting, authentication simplicity. These are the decisions that actually matter in production.

The strongest contribution is the product-trumps-design argument. Most API design advice implicitly assumes that if you build a beautiful interface, people will come. Goedecke names the uncomfortable truth: Jira's API is terrible and everyone uses it anyway, because the product has no substitute. This is the [[Designing a Passively Safe API]] counterpoint -- Albaugh shows you how to build the mechanically perfect API, and Goedecke reminds you that mechanical perfection is necessary but nowhere near sufficient.

The idempotency treatment is solid but lighter than Albaugh's. Goedecke covers the what (idempotency keys, Redis or durable storage) but not the full lifecycle (recovery points, completer/reaper processes, transient vs. non-transient error classification). Read both: Goedecke for the strategic overview, Albaugh for the implementation depth.

The GraphQL skepticism is refreshing and correct. Most API design articles treat GraphQL as a natural evolution. Goedecke treats it as a complexity tax you pay only when the flexibility is genuinely necessary -- and notes that the people who most need API simplicity (non-engineers) are the ones least equipped to write GraphQL queries.

What's missing: no discussion of API observability, no mention of contract testing, and the authentication section could go deeper on the middle ground between "long-lived key" and "full OAuth." API keys with scoping and rotation policies would be the pragmatic answer for most teams. Also absent: any discussion of how AI agents consume APIs differently than humans -- relevant given that agents are increasingly the primary API consumers, and their retry/error-handling semantics differ from human-driven code.

## Cross-links

- [[Designing a Passively Safe API]] -- the deep implementation companion. Goedecke gives the strategic overview; Albaugh gives the mechanical engineering
- [[Software Engineering Craft]] -- synthesis page covering API design, error handling, and operational reliability as enduring fundamentals
- [[Better Error Messages]] -- Goedecke doesn't cover error design, but his "boring API" philosophy implies clear, predictable error responses
- [[The Future of Software Engineering is SRE]] -- rate limiting and operational killswitches are SRE concerns wearing API clothes
- [[DAB]] -- Microsoft's auto-generated REST/GraphQL layer is the logical conclusion of "APIs should be boring": generate the boring parts
- [[Computer Use is 45x More Expensive Than Structured APIs]] -- the cost argument for well-designed APIs over vision-based alternatives
- [[Spec-First Development at Benchling]] -- define each object once, let capabilities consume the schema. The spec-first approach to Goedecke's immutability constraint
- [[Elements of Code]] -- "wrong in correctable ways" applied to API surfaces: make the API easy to use correctly, hard to use destructively

---
*Sources: [[summary/good-api-design]]*
*Last updated: 2026-05-14*
