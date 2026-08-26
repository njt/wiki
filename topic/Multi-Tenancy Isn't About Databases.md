# Multi-Tenancy Isn't About Databases

Derek Comartin argues that multi-tenancy discussions fixate on the wrong question — shared database vs. database per tenant — when the real question is what you're trying to isolate. Every shared resource couples tenants together; the architecture is a spectrum of isolation choices across data, compute, queues, caching, and deployment, not a binary database decision.

---

## Key Quotes

> "That shared infrastructure which seemed simple is also what is coupling everything together."

The core diagnosis. Simplicity at the infrastructure layer doesn't eliminate coupling — it just hides it until a tenant with different behavior exposes it.

> "Multi-tenancy is not just about databases. It is about creating isolation through boundaries. A database is just one of those boundaries."

The thesis statement. Boundaries can be drawn at the database, compute, queue, cache, or region level. The first question isn't "which database topology?" — it's "what are we actually trying to isolate?"

> "Every time tenants share something, they are coupling through it."

A structural claim: coupling isn't a design choice you opt into or out of; it's a physical consequence of shared resources. If tenants share a schema, they're coupled through migrations. If they share an API instance, they're coupled through deployments and failures.

> "There is nothing inherently wrong with this. As long as you understand the trade offs."

Comartin is no purist. A shared database with tenant ID segregation is a perfectly valid starting point. The sin isn't the design — it's not knowing what you're trading away.

> "You are providing deployment isolation. You are not necessarily providing schema isolation."

A crucial distinction. Giving Tenant A version 1 and Tenant B version 2 of your application is deployment isolation, but if they share a schema, every change must still be backward-compatible. The isolation people think they're getting from canary deployments evaporates at the data layer.

> "Sharing gives you efficiency. It is usually simpler and more cost effective. Isolation gives you more control. But that control comes with more cost and more complexity."

The fundamental trade-off, stated plainly. This isn't a moral argument — it's economics.

## Key Themes

- **#pattern** — Multi-tenancy as a spectrum, not a binary. Mix-and-match isolation at different layers.
- **#concept** — Every shared resource = coupling. This is structural, not optional.
- **#concept** — The first question: "what are you trying to isolate?" Not "which database topology?"
- **#pattern** — The tenant ID "leaks everywhere" as a trade-off, not a defect. Query filters, queue messages, cache keys — the ID propagates through the system as the price of shared infrastructure.
- **#concept** — Efficiency vs. control is the spectrum. Sharing maximizes efficiency; isolation maximizes control. Neither is correct — the right answer depends on what you value.
- **#pattern** — Tenants diverge over time. What starts as uniformity becomes stratification: small tenants pool together, large tenants get dedicated resources, noisy tenants get rate-limited or isolated.

## Critical Analysis

Comartin's framing is refreshingly honest about the trade-offs, but two things go under-explored.

**First, the operational cost of the "mix and match" approach.** A spectrum is elegant in theory but brutal to operate. If Tenant A is on a shared database but Tenant B has a dedicated one, your backup strategy, monitoring, and migration tooling now have two code paths. Add tenant-specific queue topologies, cache configurations, and deployment schedules, and you've built a combinatorial operational matrix. The article treats "mix and match" as freedom — and it is — but it's also an ops tax that compounds with every dimension of isolation you vary. The real-world answer is usually "pick two isolation tiers and force everyone into one of them," which is less elegant but more survivable.

**Second, the article underweights the organizational dimension.** The reason "shared database vs. database per tenant" dominates discussion isn't just technical myopia — it's that the database is where organizational boundaries collide. The DBA team owns the database. The platform team owns compute. The SRE team owns deployments. When you say "isolate at the queue level," you're asking the messaging team to solve a problem the database team created. Multi-tenancy discussions are political, not just architectural. Comartin's framework would be stronger if it acknowledged that the "first question" is often "who owns this problem?" and the answer isn't always the team best positioned to solve it.

That said, the article succeeds at its core mission: unbundling multi-tenancy from database choice. For anyone who's ever been in a meeting where the architecture discussion dead-ends at "well, should we go database-per-tenant?", this is the article to send before the meeting.

## Cross-References

- [[Nubase]] — Implements one point on Comartin's spectrum: physical database-per-tenant isolation via Hibernate's multi-tenancy SPI. The operational complexity of managing per-project databases validates Comartin's "control has a cost" thesis.
- [[Nango — Running Untrusted Customer Code at Scale]] — Another point on the spectrum: tenant-pinned AWS Lambda on Firecracker microVMs. The journey from shared runners to per-tenant isolation is a case study in Comartin's noisy-neighbor-driven evolution.
- [[A Convention Is Not a Constraint]] — The worked example of Comartin's "deployment isolation is not schema isolation" distinction: a django-tenants team consolidating per-tenant workers onto a shared fleet finds the enforcer was the topology, then rebuilds it as a per-tenant PostgreSQL role whose grants physically confine the connection to one schema.
- [[Software Engineering Craft]] — The hub for architecture and design fundamentals. Comartin's "what are you trying to isolate?" echoes the craft principle that first questions matter more than first answers.
- [[Databases and Data]] — The hub for storage decisions. Multi-tenancy at the database layer is one dimension of the broader storage design space these pages explore.
- [[Claude Is Not Your Architect]] — Comartin's thesis that you must *understand the trade-offs* aligns with Holland's warning against outsourcing architectural judgment. The database topology decision is exactly the kind of choice that can't be delegated to an LLM without context it doesn't have.

Related but not yet linked: rate limiting patterns, tenant-aware caching strategies, backward-compatible schema migration tooling.

---
*Sources: [[raw/multi-tenancy]]*
*Last updated: 2026-07-18*
