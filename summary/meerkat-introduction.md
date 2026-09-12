---
url: https://blog.cloudflare.com/meerkat-introduction/
title: "Introducing Meerkat: an experiment in global consensus"
author: James Larisch, Bob Halley, João Pedro Leite
date_fetched: 2026-07-11
date_published: 2026-07-08
topics:
  - databases-and-data
---

Cloudflare Research introduces Meerkat, a consensus service built on the
QuePaxa algorithm (Tennage & Băsescu et al., 2023). Unlike Raft, where only
the leader can write and leader failure blocks progress until a timeout-driven
election completes, QuePaxa lets all replicas propose writes at all times. A
leader exists but is optional — its only benefit is fewer round trips.

Meerkat targets small control-plane state (e.g., leadership for replicated
databases) across Cloudflare's 330+ data centers. It provides linearizability
and serializability, tolerating crash and network failures so long as a
majority of replicas remain reachable. Under adversarial network conditions
the algorithm reportedly sustains ~10× the throughput of Raft or Multi-Paxos.

The trade-off is latency: consensus requires 1–3 round trips to a majority
(1 for the leader, 3 for a non-leader, more under contention). Mitigations
include batching writes, stale local reads, and bundling multiple operations
into one proposal. These characteristics suit infrequently-written control
plane data that must stay consistent.

Meerkat is not yet in production. In proof-of-concept tests with up to 50
globally distributed replicas, leader failures were constant yet the cluster
continued operating with no increase in error rate. The team plans further
posts on QuePaxa internals, formal verification of their Rust implementation,
and deterministic simulation testing.
