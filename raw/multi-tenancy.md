---
url: https://codeopinion.com/multi-tenancy/
title: "Multi-Tenancy Isn't About Databases"
author: Derek Comartin
date_fetched: 2026-07-18
date_published: 2026-07-08
site: CodeOpinion
---

# Multi-Tenancy Isn't About Databases

*by Derek Comartin, published July 8, 2026 on CodeOpinion*

The article begins by describing the seemingly obvious solution to multi-tenant SaaS: Tenant A and Tenant B sharing one application and one database, with data segregated by tenant ID. The author calls this approach "simple," "easy," and functional — "until it does not" work.

The problem surfaces when one tenant imports large volumes of data or runs resource-intensive reports. "That shared infrastructure which seemed simple is also what is coupling everything together."

## Shared Infrastructure Means Shared Problems

Derek uses a rolling deployment scenario as an example: a schema change is made to the database, and one application instance is updated first. The second instance, still running old code, fails at runtime because it "has zero expectation of what that schema change is." Old and new code "are expecting a certain shape, and the database is not giving both of them what they expect."

He acknowledges blue/green and canary deployments as possibilities but stresses the need to "understand the coupling involved and the trade offs being made."

## The First Question Should Not Be Database Per Tenant

The author observes that most people immediately ask about shared database vs. database per tenant vs. schema per tenant. He argues these "should not be the first question."

A superior first question: "What are you actually trying to isolate?" — whether that's data, compute, deployments, schema changes, performance, or failure types. He states plainly: "multi-tenancy is not just about databases. It is about creating isolation through boundaries. A database is just one of those boundaries."

## Every Shared Resource Creates Coupling

"Every time tenants share something, they are coupling through it," Derek writes. In the database example, tenants share schema (requiring shared migrations) and database performance. With APIs, tenants share compute, deployments, and failures. The same logic applies to queues, background processing, caching, and even cloud regions (residency, latency). "If you are sharing something between tenants, they are ultimately coupled through it."

## A Shared Database Can Be Perfectly Fine

Derek defends the simple approach: one application, one database, tenant ID segregation. "There is nothing inherently wrong with this." It's simple, adding new tenants is easy (no new infrastructure), schema management is simpler, and it's more cost-effective. Code can use query filters for tenant scoping.

The caveat: "As long as you understand the trade offs."

## The Tenant ID Has To Leak Everywhere

The trade-off is that "this information has to leak everywhere through your system." Every query, every report, every message in a queue must carry or be filtered by tenant ID. Derek calls this "not a bad thing. It is just the trade off."

## Shared Schema Means Shared Migrations

Schema changes during rolling deployments require backward compatibility: "You expand the schema, make the change in a compatible way, deploy the new code, then backfill the data." Nullable columns help, but backfilling remains necessary. This adds complexity to the deployment pipeline. The simplicity of one physical database is maintained, but "the complexity moves into dealing with shared schema changes and rolling deployments."

## Deployment Isolation Is Not Schema Isolation

He notes you could have Tenant A on version 1 and Tenant B on version 2 of the application, providing deployment-level isolation — akin to canary deployments. However, if they share the same underlying schema, "you still have to make everything backwards compatible." He distinguishes: "You are providing deployment isolation. You are not necessarily providing schema isolation."

## Multi Tenant Architecture Is A Spectrum

The author frames multi-tenancy around isolation — not just data isolation. The spectrum includes:

- **Data isolation**: Shared tables → separate schemas → different database pools → database per tenant
- **Compute isolation**: Shared instances → pooled instances → dedicated compute
- **Queues/messaging**: Priority-based or per-tenant queues
- **Caching**: Tenant-aware cache keys and invalidation
- **Cloud regions**: Residency and latency considerations

"You do not necessarily have to fit on one side or the other. It is often a mix and match depending on your needs."

## Efficiency Versus Control

Derek presents the core trade-off as efficiency vs. control. Shared databases are simpler to maintain (one instance, one backup, one migration process). A database per tenant offers more control, but "that control is not free. There is cost and complexity." The same principle applies to compute: "Shared compute can be very efficient and cost effective" but pooled or dedicated compute introduces "more complexity involved."

## Noisy Neighbors

He illustrates the "noisy neighbor problem": Tenant B floods the system with requests (data imports, heavy reports, CPU/memory-intensive operations), degrading performance for Tenant A. This affects both the API and the database. He asks whether this justifies isolating or pooling compute differently, then suggests alternatives.

## Rate Limiting Is One Tool

Rate limiting by tenant ID is presented as a tool — but not a complete solution. "Depending on the request being made, you might save the API, but that does not mean you are saving the database." He also mentions grouping database connection pools by tenant. The real question is: "what resources are being exhausted."

## Tenants Are Not All The Same

"At the beginning, tenants might seem like they are all the same, but they probably are not." Some tenants have few users; others have many. Some run heavier workloads, import more data, or run more reports. Over time, you may need to pool small tenants together, pool large tenants together, or move very large tenants to dedicated infrastructure. "Again, it goes back to mix and match."

## When Should You Isolate?

Derek identifies four key reasons:

1. **Smaller blast radius** — when you want to limit the impact of failures
2. **Noisy neighbor prevention** — at whatever level the problem manifests
3. **Tenant-specific requirements** — such as compliance or regulatory needs
4. **Dedicated capacity** — tenants who care about availability or performance guarantees
5. **Safer rollouts** — offering one version to a subset of tenants before broader release

He summarizes: "It is isolation at various levels because you want to avoid noisy neighbors, reduce the blast radius, and have protection against specific types of failures."

## Multi-Tenancy Is About Boundaries

The concluding section drives the thesis home: "Multi-tenant architecture is not a database pattern. It is about isolation." The real question is what should be isolated. "Sharing gives you efficiency. It is usually simpler and more cost effective." Isolation gives more control but "comes with more cost and more complexity."

His final advice to anyone building or living in a multi-tenant system: "do not start with 'shared database or database per tenant?'" Instead, "start with what you are actually trying to isolate. Then decide where sharing makes sense, where isolation makes sense, and what trade offs you are willing to take."

## Related Links

- *Resilience Patterns Can Make Your System Less Resilient*
- *Stop Blaming Event-Driven Architecture*
- *Building Multitenant Systems with CosmosDB* (sponsored link)
