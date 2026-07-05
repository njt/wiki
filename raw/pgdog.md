---
url: https://pgdog.dev/
date_fetched: 2026-07-05
backfilled: true
---

// Drop-in PostgreSQL proxy, // no app changes required.

Three tools in one.

01 · Connection pooler

Real transaction mode

PgDog can share just a few Postgres connections between 100,000+ clients, without breaking Postgres features.

✓Session state: SET commands, advisory locks and LISTEN/NOTIFY; no connection pinning.

✓Multi-threaded and async: 50,000+ transactions/s per thread, without query length or connection limits.

✓Prepared statements: compatible with all client drivers, with zero overhead.

1# pgdog.toml

2[general]

3default_pool_size = 100

4

5[[databases]]

6name = "prod"

7host = "10.0.0.1"

02 · Load balancer

ALB for your database

PgDog can distribute reads between replicas, detect replication lag, hardware failure, and primary failovers. Use just one endpoint for all your queries.

✓Health checks: block out-of-date or broken replicas from serving queries.

✓Failover detection: move write traffic to the new primary, without config changes.

✓Read / write split: route SELECT queries to the replicas, all others to the primary. Uses the internal Postgres SQL parser for maximum compatibility.

1# pgdog.toml

2[[databases]]

3name = "prod"

4host = "10.0.0.1"

5role = "primary"

6

7[[databases]]

8name = "prod"

9host = "10.0.0.2"

10role = "replica"

11

12[[databases]]

13name = "prod"

14host = "10.0.0.3"

15role = "replica"

03 · Distributed database

Shard Postgres, without app changes

PgDog gets the sharding key directly from queries and sends them to the right shard. Queries without one are executed across all databases, in parallel.

✓Fast OLTP: use sharded databases just like plain old Postgres, with a horizontally scalable coordinator.

✓Fast OLAP: GROUP BY, COUNT(), AVG(), ORDER BY, MIN(), MAX(), COPY and many more ops supported out of the box. Postgres + PgDog = scatter/gather engine.

✓Shard by anything: flexible, configuration-driven data distribution, compatible with standard Postgres data types.

1# pgdog.toml

2[[databases]]

3name = "prod"

4host = "10.0.0.1"

5shard = 0

6

7[[databases]]

8name = "prod"

9host = "10.0.0.2"

10shard = 1

11

12[[sharded_tables]]

13database = "prod"

14column = "tenant_id"

Distributed, consistent, relational

PgDog doesn't trade speed for consistency. It maintains ACID compliance, at scale.

✓Cross-shard transactions: write data to all shards, atomically. Any error triggers an automatic rollback, using native Postgres prepared transactions and two-phase commit.

✓Replicated tables: not all tables need to be sharded. Store the same data on all shards and use it for fast, shard-local joins.

✓Integer primary keys: no need to switch to UUIDs. PgDog supports monotonic, big integer primary keys, generated automatically in the proxy.

✓Sharding key mutation: move rows between shards using only UPDATE statements.

✓Consistent schema: DDL is automatically cross-shard. Migration tools like Alembic, ActiveRecord migrations and others work out of the box.

1-- Cross-shard transactions,

2-- with big integer primary keys.

3BEGIN;

4

5INSERT INTO users (email, ip_addr, created_at)

6VALUES ($1, $2, now()), ($3, $4, now())

7RETURNING id;

8

9COMMIT;

10

11-- Replicated tables for shard-local joins.

12SELECT users.*, city.name FROM users

13INNER JOIN cities

14ON users.city_id = city.id

15WHERE users.id = $1

16AND users.tenant_id = $2;

17

18-- Sharding key updates.

19UPDATE users SET tenant_id = $1WHERE id = $2;

We couldn't have done it without PgDog. This was a first for me and it turned out being easier and less scary than I expected.

John Kulzick

Software engineer

Span

We're ingesting more data than ever before. Since switching to PgDog, we've only had 100% uptime.

Mike Matkiwsky

Director of Engineering

tripstack

We run thousands of pods and couldn't have scaled this far without PgDog. The team has been very responsive throughout.

Sam R.

Member of Technical Staff

Modal

PgDog has been incredibly reliable for us. It’s the kind of infrastructure that just quietly works, so we rarely have to think about it at all. Compared to RDS Proxy, it gave us a huge step up in connection scalability and removed a major bottleneck for our system.

Maher Beg

Staff Software Engineer

Ramp

Adopting PgDog let us move complicated geospatial sharding logic from the application tier into a simple config, speeding up queries in the process. We have almost 4 billion rows in production and have never seen a single issue. So glad to have PgDog breathing fresh air into the the Postgres ecosystem!
