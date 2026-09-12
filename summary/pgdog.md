---
url: https://pgdog.dev/
title: "PgDog — Scale Postgres n times"
author: PgDog (pgdog.dev)
date_fetched: 2026-05-15
date_published: unknown
topics:
  - databases-and-data
---

# PgDog — Complete Page Extraction

## Overview

PgDog is a PostgreSQL proxy tool that combines three capabilities into a single executable: a connection pooler, a load balancer, and a distributed (sharded) database system. The tagline is **"Scale Postgres *n* times."**

It deploys via Docker, Helm on Kubernetes, or as a standalone binary.

---

## Three Core Features

### 1. Connection Pooler — "Real transaction mode"
- Shares a small number of Postgres connections across 100,000+ clients without breaking Postgres features.
- **Session state**: Supports SET commands, advisory locks, and LISTEN/NOTIFY without connection pinning.
- **Multi-threaded and async**: Handles 50,000+ transactions/s per thread with no limits on query length or connections.
- **Prepared statements**: Works with all client drivers at zero overhead.

### 2. Load Balancer — "ALB for your database"
- Distributes reads across replicas while detecting replication lag, hardware failures, and primary failovers.
- Exposes a single endpoint for all queries.
- **Health checks**: Blocks outdated or broken replicas from serving queries.
- **Failover detection**: Automatically routes write traffic to a new primary without config changes.
- **Read/write split**: Routes SELECT queries to replicas; everything else to the primary, using an internal Postgres SQL parser.

### 3. Distributed Database — "Shard Postgres, without app changes"
- Extracts the sharding key directly from queries and routes to the correct shard. Queries without a sharding key execute across all databases in parallel.
- **Fast OLTP**: Sharded databases behave like plain Postgres with a horizontally scalable coordinator.
- **Fast OLAP (scatter/gather)**: Supports GROUP BY, COUNT, AVG, ORDER BY, MIN, MAX, COPY, and more out of the box.
- **Shard by anything**: Configuration-driven data distribution compatible with standard Postgres data types.

### Distributed Database Advanced Features
- **Cross-shard transactions**: Writes to all shards atomically; errors trigger automatic rollback via Postgres prepared transactions and two-phase commit.
- **Replicated tables**: Non-sharded tables stored on all shards for fast local joins.
- **Integer primary keys**: Supports monotonic big integer primary keys generated in the proxy (no UUID requirement).
- **Sharding key mutation**: Rows can move between shards using UPDATE statements.
- **Consistent schema**: DDL runs automatically across all shards; migration tools like Alembic and ActiveRecord work without changes.

---

## Architecture

The visual architecture shows: **Your App** → **PgDog** (which acts as pooler + load balancer + distributed SQL coordinator/sharder) → **PostgreSQL instances** (primaries, replicas, and shards).

The config-driven setup uses a `pgdog.toml` file where you define databases with host, port, role (primary/replica), and shard assignments.

### Sharding Model (visualized)
- **Shard 0**: tenants 0–33%
- **Shard 1**: tenants 34–66%
- **Shard 2**: tenants 67–100%

Each shard appears to have one primary + 2 replicas.

---

## Deployment Options

| Method | Command |
|--------|---------|
| **Helm (Kubernetes)** | `helm repo add pgdogdev https://helm.pgdog.dev` then `helm install pgdog pgdogdev/pgdog` |
| **Docker** | `docker run ghcr.io/pgdogdev/pgdog:latest` |

---

## Performance Metrics (claimed)

- 10 TB+ sharded in production
- 1 M+ queries/second in production

---

## User Testimonials / Use Cases

| Source | Quote (trimmed to 125 chars) |
|--------|------------------------------|
| **John Kulzick, Span** | "We couldn't have done it without PgDog. This was a first for me and it turned out being easier and less scary than I expected." |
| **Mike Matkiwsky, TripStack** | "We're ingesting more data than ever before. Since switching to PgDog, we've only had 100% uptime." |
| **Sam R., Modal** | "We run thousands of pods and couldn't have scaled this far without PgDog. The team has been very responsive throughout." |
| **Maher Beg, Ramp** | Compared to RDS Proxy, PgDog provided "a huge step up in connection scalability and removed a major bottleneck." |
| **Hutch Ingold, Earth Genome** | Moved "complicated geospatial sharding logic from the application tier into a simple config" with nearly 4 billion rows and zero issues. |

## Known Users (Logos)

Coinbase, Bitstack, Reducto, Circleback, TripStack, Span, Ramp, Modal, Huntress, Earth Genome

---

## Client Drivers Mentioned

asyncpg, pgx, libpq, ruby-pg

---

## Docs & Resources

| Section | Links |
|---------|-------|
| **Product** | Open source docs, Enterprise page, Blog |
| **Docs** | Installation, Connection pooler, Load balancer, Sharding |
| **Support** | GitHub (4.3k stars), Discord, Calendly (talk to us), Email |

---

## Pricing

No pricing information is visible on this page. The product has an open-source offering (docs.pgdog.dev) and a separate **Enterprise** page (linked but not detailed here). The footer also links to a Privacy policy and Terms of service.

---

## Key Takeaway

PgDog is positioned as a drop-in PostgreSQL proxy requiring no application code changes — it replaces or augments tools like RDS Proxy while adding sharding capabilities typically found in middleware like Citus, but with a simpler configuration-driven approach.
