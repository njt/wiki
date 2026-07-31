# Session Revocations at Scale (Canva)

How Canva rebuilt its session revocation cache using S3 as a durable distribution layer, a hand-rolled 16-byte binary format, and optimistic concurrency — cutting memory footprint 8× while eliminating the deployment-time "coordinated stampede" on MySQL.

---

## The Problem

Canva serves hundreds of millions of users, with every backend request identifying the logged-in user hundreds of thousands of times per second. They use encrypted browser cookies so gateways can trust user identity, permissions, and roles without a datastore lookup on every request. But when a user is logged out or permissions change, those revocations must propagate to every gateway in near real time.

The old system: each gateway pulled over a million revocations from MySQL on startup. During deployments, hundreds of pods stampeded the database simultaneously. The fix was temporary — add read replicas — and the problem kept growing.

## The Architecture

### S3 as a Cache Distribution Layer

The team rejected Redis: not typically deployed in a fully durable configuration, and a Redis cluster plus consistency management was more complexity than they wanted. Instead they chose **S3** — "designed to serve downloads of large, durably stored files at a low cost," the article notes. This is the key inversion: S3 is not a cache, but it's a better cache distribution mechanism than a cache for this workload.

> "We use in-memory lookups because they're faster and more reliable than checking a networked datastore."

The gateways hold the cache in memory. S3 is just the distribution pipe. The lookup is local; the download is cheap and infrequent.

### Sliding Window, 30-Minute Chunks

Revocations are partitioned into 30-minute segments, each one S3 object. Chunk names encode the start timestamp, enabling efficient sorted-key lookups. Gateways download only the most recent 12 hours of chunks (all tokens refresh within this window).

Conditional GET requests redownload only changed chunks. Even 1 million revocations in a 30-minute window is just 16 MB, polled a few times per minute — negligible versus the gateway's proxy traffic.

### 16-Byte Binary Format, 8× Memory Reduction

Each revocation packs a principal and a login timestamp cutoff into 16 bytes via bit twiddling. Chunks are flat, sorted arrays that gateways operate on directly with binary search — no deserialization, no Java object overhead.

> The memory footprint was reduced by 87.5% versus the prior multi-Java-object representation.

Multiple revocation types are supported by reserving flag bits (targeting a brand, invalidating cached cookie info without full logout) while still sorting by principal.

### Optimistic Concurrency, Not Distributed Consensus

An async worker scans the database for new revocations, fetches the latest chunk from S3, inserts into the sorted array, and re-uploads. Conditional PUT requests assert the chunk hasn't changed since read.

> "The PUT conditions guarantee that each read-modify-write operation only ever appends to the set of revocations in S3."

ZooKeeper leader election reduces conflicts, but correctness doesn't depend on it. A paused node that wakes up and overwrites a new leader's writes can't violate the append-only invariant — the conditional PUT fails, and it retries.

## Key Themes

#pattern The **S3-as-cache-distribution** pattern: use a durable object store as the distribution layer for in-memory caches across a fleet. Reject Redis when durability and simplicity matter more than latency.

#pattern **Append-only with optimistic concurrency**: conditional PUTs as a correctness primitive that's cheaper and simpler than distributed consensus. The invariants are enforced at the storage layer, not the coordination layer.

#pattern **Binary format as optimization**: 16 bytes per record, sorted flat arrays, binary search, zero deserialization. An 8× memory reduction that came from treating the data as bytes, not objects.

#concept **Scaling in theory vs. practice**: the worker is bottlenecked on network latency, not on sorting dense arrays. "Even an unoptimized implementation can achieve a write throughput above 2,000 revocations per second." The O(N²) chunk-building algorithm isn't the bottleneck — the network is.

## Critical Analysis

**The S3-over-Redis choice is the most interesting decision here, and the article underplays it.** Redis is exactly what most teams would reach for — a fast, networked cache. But Canva recognized that the durability requirements (revocations must survive restarts) and the read pattern (bulk download, not point queries) made S3 the better fit. This is taste: knowing when the obvious tool is the wrong tool.

**The 16-byte binary format is a flex, and it works.** An 8× memory reduction isn't marginal — it changes what fits in a pod's memory budget. But the real win isn't the bytes saved; it's that gateways operate directly on downloaded bytes with no transformation. The format is the API. This eliminates an entire class of bugs (deserialization errors, version mismatches, object allocation pressure).

**The "scaling bottleneck is network latency" finding is the most honest sentence in the article.** Every distributed systems engineer has a story about the clever algorithm they wrote that was 10× faster than needed, while the real bottleneck was something mundane they didn't measure first. Vallis admits it: "our worker implementation is actually bottlenecked on network latency." This is the article's most valuable contribution — not the architecture, but the humility to report what the bottleneck actually was.

**The coordinated stampede is the silent killer of deployment velocity.** When every pod races to the database on startup, you can't deploy fast. Moving the cache bootstrap from MySQL to S3 (where the blast radius is the storage service, not your own database) made deployments faster and reduced read replicas from many to two. This is the kind of win that shows up in incident frequency and on-call happiness, not in a benchmark.

## Connections

[[Distributed Systems]] — This is a case study in the hub's themes: cache consistency without distributed consensus, append-only invariants, and the practical gap between what you design and what bottlenecks.

[[Software Engineering Craft]] — The article models good engineering taste: reject the obvious tool (Redis), choose the boring one (S3), measure before optimizing, and report honestly what you found.

[[Queues Don't Fix Overload]] — Fred Hebert's 2014 argument that queues treat symptoms, not causes. Canva's solution is the counterexample done right: they identified the bottleneck (coordinated MySQL stampede), then moved it to a system (S3) designed for that load pattern.

[[Your Distributed System Is Slower Than a Laptop]] — The COST paper's thesis that distribution tax exceeds single-machine performance applies here in reverse: Canva found a single-machine pattern (in-memory binary search over sorted arrays) and scaled it with cheap S3 distribution rather than building a distributed cache.

[[Signals — The Push-Pull Algorithm]] — A different domain, but the same insight: eager invalidation + lazy evaluation. Gateways download revocation chunks lazily (conditional GETs, polled) but check them eagerly (every request against in-memory cache).

[[Aurora DSQL]] — AWS's serverless active-active SQL database with commit-time optimistic concurrency. Canva's conditional PUT pattern is the object-store analogue of the same idea: let the storage layer enforce consistency, not the application.

---

*Sources: [[raw/session-revocations-at-scale]]*
*Last updated: 2026-08-01*
