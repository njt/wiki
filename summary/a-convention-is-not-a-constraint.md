---
url: https://www.simplethread.com/a-convention-is-not-a-constraint/
title: "A Convention Is Not a Constraint"
author: Simple Thread
date_fetched: 2026-08-26
date_published: n.d.
topics:
  - software-engineering-craft
---

A field report from Simple Thread on rebuilding multi-tenant isolation after
consolidating per-tenant worker deployments onto a single autoscaled fleet. The
thesis, stated twice for emphasis: a convention is only as strong as everyone's
discipline in following it; a constraint is enforced whether anyone remembers it
or not.

The concrete setup is a Django product on django-tenants — one PostgreSQL
database, many schemas — running long, compute-heavy jobs via dramatiq. In the
first multi-tenant architecture, isolation for background work lived in the
deployment topology: each tenant had its own workers and queues, so a worker
could never touch another tenant by construction. Nothing in the code enforced
it, and nothing needed to. When the team moved to a shared worker pool to make
onboarding cheap and compute elastic, that enforcer disappeared by design.

The hazard is a worker process reusing one database connection across tenants:
a stale `search_path`, a pooled connection still authenticated as the previous
tenant, or a teardown skipped by an early exception — any of them a silent
cross-tenant data leak. The fix is a `@tenant_task` decorator that stamps the
tenant schema onto each queued message and, per task, sets the connection up and
tears it down in a `finally` block. Two layers of defense follow:

- **Layer one (correctness):** set `search_path` to the tenant's schema, reset
  it afterward. Correct exactly as long as the code is correct.
- **Layer two (confinement):** connect as a per-tenant PostgreSQL role
  (`{schema}_user`) granted access to only that tenant's schema, so a
  schema-qualified query reaching past the role dies on a permission error, and
  a task that forgets the decorator is stranded in `public` with nothing
  tenant-shaped to read.

The honest limitation: layer two does not check that the *right* tenant was
stamped upstream — it bounds what a task can do given that decision, and the
trust boundary is the same django-tenants request resolution the web tier
already relies on. The article also lays out the isolation ladder (database per
tenant → deployment per tenant → schema per tenant with runtime context →
shared schema with tenant filtering) and the two "fail closed" paths: a missing
`tenant_schema` raises, and a missing decorator hits a role that can reach
nothing.
