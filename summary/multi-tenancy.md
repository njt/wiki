---
url: https://codeopinion.com/multi-tenancy/
title: "Multi-Tenancy Isn't About Databases"
author: Derek Comartin
date_fetched: 2026-07-18
date_published: 2026-07-08
---

Derek Comartin argues that multi-tenancy discussions start in the wrong place. The
first question should not be "shared database or database per tenant?" but "what
are you actually trying to isolate?" A database is just one boundary among many.

Every shared resource couples tenants through it — database schema and
performance, API compute and deployment, queues, caches, even cloud regions. A
shared-database approach with tenant ID segregation is perfectly fine to start
with, but the trade-off is that the tenant ID must leak everywhere (queries,
reports, queues). Schema changes on shared infrastructure demand backward
compatibility, adding deployment complexity.

The core tension is efficiency versus control. Sharing is simpler and cheaper;
isolation gives finer-grained control but costs more. The author frames
multi-tenancy as a spectrum across data, compute, messaging, caching, and
regions — you mix and match rather than picking one side.

Key reasons to isolate: smaller blast radius, noisy-neighbor prevention,
tenant-specific compliance needs, dedicated capacity guarantees, and safer
rollouts. Rate limiting helps but isn't a complete answer — it protects the API
but not necessarily the database. The practical advice: accept that tenants
diverge over time, expect to pool some and dedicate others, and start the
conversation with boundaries, not databases.
