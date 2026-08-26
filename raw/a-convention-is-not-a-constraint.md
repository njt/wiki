---
url: https://www.simplethread.com/a-convention-is-not-a-constraint/
date_fetched: 2026-08-26
---

Every multi-tenant application makes a promise: Tenant A will never see Tenant B’s data. And in every multi-tenant application, *something* is responsible for keeping that promise. My question is one I think every team should be able to answer quickly and out loud: **what is that something?** Point at it. Name the specific mechanism that stops the leak.

If you can’t, that’s worth worrying about. Not because you’re necessarily leaking data today, but because a promise nothing is consciously keeping is a promise being kept by accident. And accidents are exactly the kind of thing a future change quietly takes away.

The application I’ll use to make this concrete is a Django product that runs long, compute-heavy jobs on each customer’s data. It started life serving a single customer, so for a while the question didn’t arise. When we made it multi-tenant, our first architecture answered the question with the deployment itself: each tenant got its own workers, and a worker could only ever touch the one tenant it was deployed for. That’s a real answer, and a good one. But notice where it lives. It’s enforced by the shape of the deployment, not by anything you could point to in the code. Later, when we consolidated onto a single autoscaled fleet so we could add tenants without standing up new infrastructure each time, that enforcer went away by design, and the guarantee it had been providing had to be rebuilt somewhere we controlled directly. This post is about that move: recognizing what had quietly been doing the enforcing, and putting it back one layer down where we could see it.

## The setup: one database, many schemas

The application is multi-tenant on top of django-tenants, which gives every customer its own PostgreSQL schema inside a single shared database. A `Client` model maps to a schema; a `Domain` model maps a hostname to a client. On every web request, middleware reads the Host header, resolves the tenant, and sets PostgreSQL’s search_path to that tenant’s schema for the life of the request. So `tenant-a.example.com` resolves to the tenant_a schema, and every ORM query inside that request transparently hits the right one. This part is well-trodden. It’s most of what you’ll find if you search “Django multi-tenancy,” and for the request/response path the enforcer is easy to name: that middleware, running on every request. The interesting part of this post is everything that happens when there is no HTTP request at all.

## Where isolation used to live: the deployment itself

The application does a lot of heavy lifting outside the request/response cycle. A single job can run for minutes to hours, so the real work happens in background tasks pulled off a queue by worker processes. We use dramatiq for this, and the code below is written against its API, but nothing in the reasoning depends on that choice. Swap in Celery or any other task queue and both the problem and the shape of the fix are the same: a shared pool of workers running tasks for many tenants through a reused database connection.

In our first multi-tenant architecture, **each tenant had its own pool of workers and its own dedicated queues**. A given worker process was launched for, and only ever served, one tenant. It pulled from that tenant’s queues, ran that tenant’s tasks, and connected to the database as that tenant.

The thing worth noticing: *nothing in our code enforced isolation for background work, and nothing needed to*. A worker never had to ask “whose task is this?” The answer was implicit in which deployment it belonged to. There was no code path by which it could touch the wrong tenant’s schema, because it had no knowledge of, and no queue subscription to, any other tenant. The enforcer was the topology, and a topology leaves no trace inside the code: you will not find a line you can point to and say “this is what keeps tenants apart,” because there wasn’t one. It was safe, and it asked nothing of the codebase.

Hold onto that last point. A guarantee that lives entirely outside the code is exactly what matters once the architecture changes.

## Why that couldn’t last: it didn’t scale with tenants

The dedicated-everything model has a problem that has nothing to do with correctness: **it doesn’t scale with the number of tenants**.

Onboarding a new customer meant standing up a new set of workers and queues and provisioning capacity for them. It was an infrastructure project, not a database row. Worse, the capacity was rigid: a quiet customer’s reserved workers sat idle while a busy customer’s queue backed up right next door, and there was no way to lend one’s headroom to the other. The per-tenant overhead of this architecture grows linearly with every new customer, in both money and manual setup.

We wanted the opposite property. We wanted onboarding a tenant to be cheap and fast, and we wanted compute to flow to wherever the demand actually was. That means one worker deployment, not N deployments. So we moved the workers onto a single autoscaling deployment and **unified the pool**: one set of workers pulling from shared queues, scaling up and down against total demand regardless of which tenant created the work. Adding a tenant stopped being a capacity decision. This is the standard consolidation most teams make as they grow, and for our scalability priority it was unambiguously the right move.

It also meant giving up the isolation enforcer we’d been relying on. That was a known consequence, not a surprise. But knowing it was coming is exactly what put the next problem squarely on our plate: the guarantee the topology used to provide for free now had to be built by hand.

## The hazard we’d just created

Here’s the new reality. A single worker process pulls Tenant A’s task off a shared queue and runs it. It finishes, and the very next thing it does is pull Tenant B’s task off the same queue and run it, all** in the same process, reusing the same database connection**.

The guarantee we used to get from the topology is simply gone. There is no longer any structural reason a worker can’t cross tenants; in fact it’s now expected to serve all of them, one after another. Isolation has to be actively established on every single task and torn down again afterward. And the failure modes are exactly the kind that don’t show up in a quick test:

- A `search_path`left pointing at Tenant A’s schema when Tenant B’s task starts.
- A pooled connection still authenticated as the previous tenant.
- An exception that bails out of a task before its cleanup runs, leaving the connection in a dirty state for whatever runs next.

Any one of those is a silent cross-tenant data leak. Not a crash, not an error, just one customer’s task quietly reading or writing another customer’s data. For us, that’s about the worst bug we could ship.

## A quick aside: where can isolation live?

Before I show you what we did, it’s worth stepping back, because “re-establish tenant context per task” is one option on a spectrum, not the only answer. Roughly, from most isolated to least:

- **A database per tenant.**The strongest separation and the heaviest operational cost. The enforcer is the connection string.
- **A deployment per tenant.**Where we started. The enforcer is the topology: safe, invisible, and expensive to scale with tenant count.
- **A schema per tenant, with context set at runtime.**Where we are now. One shared fleet; the enforcer is whatever sets and resets the tenant context around each unit of work.
- **A shared schema with a tenant column, filtered in the application (or via row-level security).**The lightest to operate and the easiest to get catastrophically wrong, because a single missing- `WHERE`clause is a leak.

None of these is universally correct. You’re picking where the isolation bill gets paid: in operational overhead, in application discipline, or somewhere in between. We landed on schema-per-tenant with runtime context because we’d already built on it for the web path, and because it let us have the one autoscaled fleet we needed without dropping all the way down to application-level filtering. The point of laying the ladder out is this: once you’ve picked a rung, you should know exactly what on that rung is doing the enforcing. On ours, we had to build it.

## Rebuilding the guarantee, one layer down

The job is to make every task execution responsible for both establishing and dismantling its own tenant context. The schema name has to ride along with the work, so when we enqueue a task we stamp the current tenant onto the message explicitly:

```
# at enqueue time, in a view that has a request
tenant_schema = request.tenant.schema_name
Message(
  actor_name="run_study",
  kwargs={
    "task_id": task_id,
    "study_id": study.id,
    "tenant_schema": tenant_schema,
},
# ...
)
```
Then, inside the worker, something has to read that schema off the message and set up the context before the task’s real code runs, then guarantee the teardown afterward. You could hang that off your task framework’s middleware, so individual tasks never see it. We chose a decorator instead, which keeps the tenant an explicit, visible part of each task: with `tenant_schema` right there in the message, we can look at any queued or retried job in the broker and see exactly whose it is. Here’s the shape of it:

```
def tenant_task(func):
  @wraps(func)
  def wrapper(*args, **kwargs):
    schema_name = kwargs.get("tenant_schema")
    if not schema_name:
      # Fail closed
      raise ValueError("Missing tenant_schema")
    # Default user only has `public` schema access
    default_user = connection.settings_dict["USER"]
    try:
      # Layer 2: connect as a role that *physically cannot*
      # reach any other tenant's schema.
      connection.settings_dict["USER"] = f"{schema_name}_user"
      connection.close() # force a fresh connection as the new user
      # Layer 1: point search_path at this tenant's schema.
      connection.set_schema(schema_name)
      return func(*args, **kwargs)
    finally:
      # Always tear it down, even on failure, so the *next*
      # task on this worker doesn't inherit our context.
      connection.settings_dict["USER"] = default_user
      connection.close()
      connection.set_schema_to_public()
return wrapper
```
There are two layers of defense here, and they guard against different things. That’s the part I want to be precise about, because it’s easy to oversell.

**Layer one is correctness.** `connection.set_schema(schema_name)` sets search_path so the ORM queries the right tenant’s schema, and the finally block tears it down so the next task on this recycled worker starts clean. If everything works, this is enough. But look at how much weight “if everything works” is carrying: a search_path that drifts partway through a task, a teardown that doesn’t run because something bailed early, a stray query that reaches into a schema it shouldn’t. Layer one is correct precisely as long as the code is correct. Put another way, the guarantee is only as good as our vigilance. In a system where a mistake means a data breach, that should make you uncomfortable. It made me uncomfortable.

**Layer two is what lets me sleep, though not for the reason I first assumed.** Two things are true about how our workers talk to the database. First, they connect by default as a PostgreSQL user granted access to the public schema and nothing else, with no tenant schema at all. Second, when we provision a tenant, we create a dedicated role, {schema}_user, granted access to only that tenant’s schema (plus public):

```
# tenant provisioning, lightly trimmed
cursor.execute(f"CREATE ROLE {username} WITH LOGIN PASSWORD %s", [password])
cursor.execute(f"GRANT USAGE ON SCHEMA {schema_name} TO {username}")
cursor.execute(f"GRANT USAGE ON SCHEMA public TO {username}")
cursor.execute(f"GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA {schema_name} TO {username}")
```
The decorator switches the connection to that role for the duration of the task, and switches it back afterward. That gives us two guarantees that hold no matter what the task’s own code does:

- **Confinement.**- `search_path`only decides how unqualified table names resolve. The moment a query names a schema explicitly — a hand-written- `SELECT ... FROM tenant_b.results`, a raw c- `ursor.execute`with an interpolated schema name, a reporting query that reaches across schemas —- `search_path`drops out of the picture entirely and PostgreSQL goes straight where the query points. Layer one has nothing to resolve, so it offers no protection at all. Layer two still does: the connection is authenticated as- `{schema}_user`, a role with no USAGE on any schema but its own, so that cross-schema query dies on a permission error instead of quietly returning another tenant’s rows. The blast radius of a bug inside a task is bound to the single tenant it declared, never some arbitrary other tenant, and never all of them.
- **No context, no access.**The default connection is authenticated as a user granted- `public`and nothing else, and no tenant’s tables live in- `public`. So forgetting the @tenant_task decorator doesn’t hand the task a silent fallback to some tenant’s data or leftover state from the previous one; it strands the task in- `public`, where there is nothing tenant-shaped to read. An ordinary query for a tenant table finds no such table and errors out, and a query that names a tenant schema directly is denied outright, since this user has no rights there. Absence of tenant context isn’t papered over with real data; the database turns it into a loud failure.

Now the honest part, and it’s a crucial point to state explicitly: **Layer two does not check that we picked the  right tenant.** The role name (

`{schema}_user`) and the `search_path` are both derived from the same `tenant_schema` value carried on the message. If the wrong schema were stamped onto a task upstream, both layers would faithfully point at that same wrong tenant, and PostgreSQL would happily allow it. That’s a different failure from the one confinement catches: a schema-qualified query reaching into some other tenant is stopped cold by the role’s grants, because then the role is right and the query is reaching past it. What Layer two can’t catch is the message naming the wrong tenant in the first place, because then the role itself is the wrong one. Which tenant a task belongs to is something we trust `django-tenants` and the initiating request to determine. It’s the very same mechanism the entire web tier already relies on to resolve `request.tenant`. Layer two doesn’t second-guess that decision; it bounds what a task can do given that decision.That distinction is the real lesson of the post, so I’ll state it precisely:

a convention is only as strong as everyone’s discipline in following it; a constraint is enforced whether anyone remembers it or not. So be clear about which of your guarantees is which.


“We always set the correct schema, and always tear it down” is a convention. “This connection physically cannot touch more than one tenant, and touches none without explicit context” is a constraint. The convention decides *which* tenant; the constraint guarantees the answer is always exactly one: never leftover state, never all of them, never `public` by accident. When you take isolation out of the topology, that constraint is the part worth rebuilding as a real database-enforced boundary, precisely because it keeps holding on the day the convention slips.

A couple of smaller decisions in that decorator matter more than they look:

- Fail closed, twice. A task whose message is missing tenant_schema raises before it runs. An ambiguous tenant context is treated as a bug, not defaulted away. That’s the application-level twin of the public-only database user above: one path catches a missing value when the decorator is present, the other catches a missing decorator with a user that can’t reach anything. Both turn “you forgot something” into a hard stop instead of a silent default.
- The `finally`block is non-negotiable. Establishing context is easy; the reason this is a runtime invariant and not a one-liner is that you must guarantee teardown (reset the user, drop the connection, reset the schema) so the next tenant’s task on this recycled worker starts clean no matter how the previous one ended.

## The takeaway

So: where does your tenant isolation live? In our first multi-tenant architecture, ours lived in the deployment topology: a real guarantee, but one enforced by infrastructure rather than by anything in the code. Consolidating onto a shared fleet was the right move for scale, and it relocated that guarantee whether we wanted it to or not. An isolation boundary that used to be structural became something the codebase had to uphold on every unit of work. A guarantee with no line of code behind it is easy to lose track of exactly when you change the thing providing it. That’s why the first useful step, before we flipped the architecture, was to say out loud what had been doing the enforcing.

Most growing teams make that move eventually. When you do, do the part that’s easy to skip: name the invariant out loud, enforce it on every task, and back the convention with a constraint the system will keep for you even on the day your code doesn’t, the way a per-tenant database role backs our search_path. Make it so that the day you’re wrong is not the day your customers find out.

Loved the article? Hated it? Didn’t even read it?

We’d love to hear from you.
