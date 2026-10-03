---
url: https://codeopinion.com/multi-tenant-best-practices-can-backfire/
title: "Multi-Tenant Best Practices Can Backfire"
author: Derek Comartin
date_fetched: 2026-10-03
topics:
  - software-engineering-craft
  - databases-and-data
---

Derek Comartin (CodeOpinion) opens with a horror story: a multi-tenant system where an HTTP API puts an invoice-generation message on a queue, a background worker processes it — no exceptions, no errors — but it executed for tenant B instead of tenant A. The root cause is a design decision made to avoid "littering" the tenant ID everywhere: an implicit tenant context established once per process, which doesn't survive the hop from HTTP request context to message-processing context.

The essay's core move is to reframe every multi-tenancy "best practice" as a tradeoff rather than a rule. Explicit tenant identity (the tenant ID flowing through requests, messages, logs, and database operations) is noisy but visible; implicit tenant context is convenient but hides a dependency that must be rebuilt at every execution-context boundary. Automatic tenant filtering (global query filters, row-level security) gives a safer default but forces an escape hatch when reporting, analytics, or admin genuinely needs to cross tenant boundaries — a dangerous exception to your own rule. Database-per-tenant buys isolation but makes cross-tenant aggregation a structural problem. Feature flags versus modularized tenant-specific features is the same story at the application level: conditional complexity in one codebase, or structural complexity in deployments and versioning.

The unifying claim: "You're not removing complexity. You're moving it." Comartin despises one-right-way prescriptions (pass the tenant ID / never pass it, use / don't use global query filters) and replaces them with a question set: where does tenant identity come from, where should it be explicit, where is it safe for it to be implicit, what happens at execution-context boundaries, and where are you willing to accept complexity? Explicit gives visibility; implicit gives convenience — the same tradeoff as shared infrastructure versus isolation.
