# Multi-Tenant Best Practices Can Backfire

Derek Comartin's essay argues that every multi-tenancy "best practice" — implicit tenant context, global query filters, database-per-tenant, feature flags — trades one problem for another rather than solving it. Its centerpiece is a silent bug where an invoice message processed for the wrong tenant because the implicit tenant context didn't survive the hop from HTTP request to background queue consumer. The lesson generalizes far beyond tenancy: you don't remove complexity, you choose where it lives.

---

## The bug that motivates it

The opening story is the strongest part. An HTTP API enqueues an invoice-generation message; a background process consumes it; nothing fails — no exceptions, no failed queries, no log errors. But it billed tenant B for tenant A's work. The culprit: the team decided the tenant ID shouldn't be "littered everywhere," so they established a tenant context once per execution. The context of an HTTP request and the context of a queue consumer are completely different, and nothing forces you to rebuild it.

> "The context of an HTTP request and the context of processing a message from a queue in a separate background process are completely different."

This is failure-by-absence: the system worked exactly as designed, which is why nothing screamed. Implicit context fails quietly; explicit data fails loudly.

## Explicit versus implicit, everywhere

Comartin runs the same tradeoff across four design axes:

- **Tenant ID everywhere vs. tenant context.** Explicit identity is "visible noise" — repetitive, but the dependency is obvious. Implicit context makes APIs cleaner but hides a dependency that "has to exist" and must be rebuilt across execution boundaries.
- **Automatic filtering vs. escape hatches.** Global query filters and row-level security give a safer default ("you don't want every query to require you to remember to add another filter"), but reporting, analytics, and administration eventually need to cross tenant boundaries — so you build an escape hatch, "an exception to the rule. That exception can be dangerous, but you still need it."
- **Shared database vs. database per tenant.** Isolation versus the structural problem of aggregating across databases for cross-tenant reporting.
- **Feature flags vs. modules.** Conditional complexity inside one application versus structural complexity across deployments, versioning, and boundaries — the moment you ask whether you have "multiple products inside one codebase."

Each row of the table is the same sentence with different nouns.

## The thesis

> "You're not removing complexity. You're moving it."

This is the essay's whole payload, and it's stated with refreshing bluntness. Comartin "despises the idea that there's one right way to build a multi-tenant system" and rejects both sides of each standard argument ("use global query filters" / "don't") because they skip the only interesting question: **what are you trading, and where are you willing to accept complexity?** He closes with a question set — where does tenant identity come from, where should it be explicit, where is it safe to be implicit, what happens at execution-context boundaries — that reads as a usable design checklist.

## Analysis

The argument is correct but risks being mistaken for relativism. "It's all tradeoffs" is cheap unless you can say *which* complexity is cheaper where — and Comartin mostly can. The queue bug isn't a symmetrical tradeoff; implicit context across a process boundary is simply wrong, not merely costly, because correctness there is invisible until it fails. The honest reading is that some placements of complexity are unpayable: if a context must be rebuilt at every async boundary, "convenient" is a lie.

There's also a sharper framing he only gestures at: implicit context is a form of ambient state, and ambient state is exactly what fails under concurrency and re-entry. The essay would be stronger if it named the pattern rather than treating "explicit vs. implicit" as a fresh dichotomy. Still, as a corrective to best-practice-listicle thinking, it does the job — the escape-hatch observation in particular is the least-discussed cost of automatic filtering.

## Connections

- Strengthens [[Multi-Tenancy Isn't About Databases]]: both insist tenancy is an application-design concern, not a database-schema decision; this adds the execution-context-boundary failure mode and the escape-hatch cost of automatic filtering.
- Nuances [[Queues Don't Fix Overload]]: a different queue lesson — the danger isn't only load, it's context discontinuity; anything ambient you established on the HTTP side dies at the enqueue boundary.
- Complicates [[Systems Ideas That Sound Good]]: a concrete worked example of its thesis — automatic tenant filtering is a sound idea whose second-order cost (the escape hatch) is where the real design lives.
- Echoes [[Locality of Behaviour]]: Gross's "behaviour obvious on inspection" and Comartin's "visible noise" are the same aesthetic — explicit dependencies you can see beat implicit ones you must know about.

---

*Sources: [[raw/multi-tenant-best-practices-can-backfire]], [[summary/multi-tenant-best-practices-can-backfire]]*
*Last updated: 2026-10-03*
