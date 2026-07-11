---
url: https://blog.cloudflare.com/meerkat-introduction/
title: "Introducing Meerkat: an experiment in global consensus"
author: James Larisch, Bob Halley, João Pedro Leite
date_fetched: 2026-07-11
date_published: 2026-07-08
---

# Introducing Meerkat: an experiment in global consensus

*By James Larisch, Bob Halley, João Pedro Leite — July 8, 2026*

## The Problem

Many internal Cloudflare services must read and modify control-plane state across the company's 330+ global data centers. These services need strong consistency—guaranteeing that "different readers *never* see inconsistent state"—and must remain available for writes even when data centers or links fail.

The Internet is unpredictable: servers crash, queues fill, cables get cut. These conditions make it difficult to run globally available data systems that guarantee linearizability (the property that all reads after a write will see that write).

## Consensus Algorithms

A *consensus algorithm* allows machines to agree on a sequence of values (like KV store operations) as long as a majority remains alive. However, commonly deployed algorithms like Raft rely on leaders and timeouts. The leader is the only replica permitted to make writes. If the leader fails, the system becomes unavailable until another replica times out and a new leader is elected. Configuring these timeouts is hard in networks with unpredictable latencies. Cloudflare has "experienced multiple incidents caused by unavailable leaders in consensus-driven systems."

## Meerkat and QuePaxa

For the past year, Cloudflare Research has been building **Meerkat**, a consensus service powered by the **QuePaxa** algorithm (published in 2023 by Tennage & Băsescu et al.). The key differentiator: in QuePaxa, "all replicas can perform writes at all times, and progress is never halted due to a timeout."

Meerkat is designed initially for managing small control-plane state (e.g., leadership for replicated databases) and will remain internal-only for the near future.

## Requirements

**Strong consistency:** Cloudflare services want linearizability—the ability to reason about distributed data like local memory on a single-threaded machine. Meerkat's KV store additionally provides serializability.

**Fault tolerance:** The system should remain available as long as a majority of machines are alive and a client can contact any machine connected to that majority. Correctness is maintained even with crashes, restarts, and network failures, but Byzantine faults are not handled.

## Architecture

Developers request a cluster of Meerkat replicas. Each replica connects to every other, participates in consensus, and can receive reads and writes. Clients send app-specific requests to any replica, which translates them into *log events* distributed across all replicas using consensus. All functioning replicas maintain the exact same log.

The log is a sequence of slots. All slots except the last are "decided." An invariant: if any two replicas decide on a value for a slot, those values are identical. When a client submits a `put`, the receiving replica triggers consensus for the next empty slot. If two replicas propose different values for the same slot, the algorithm ensures only one wins out—at least a majority agrees, and the non-majority can "never decide a different proposal."

This ensures linearizability: even if a reader contacts a replica that missed the latest write, that replica cannot complete the read without first learning about the already-decided slot, thus ordering the read after the write in the log.

## How Meerkat Outperforms Raft

Raft's authoritative leader creates two problems:

1. If the leader goes down, all writes block until a new leader is elected.
2. If the leader slows down (overloaded or network delays), performance degrades—it's a bottleneck.

Leader election uses timeouts, which are problematic in wide-area networks. If timeouts are shorter than network delay, replicas constantly time out. If too long, the system reacts slowly to failures. Concurrent leadership campaigns can also interfere destructively.

QuePaxa avoids this. A client contacts any replica to drive consensus. There is a leader, but it's optional—its only advantage is fewer round trips (one instead of three+). Clients can contact multiple replicas concurrently for the same proposal without destructive interference; "replicas *work together* to decide one of the proposed values."

The three key advantages: no single-replica failure point, no degrading leader elections, and QuePaxa was designed for asynchronous networks with adversarial conditions—maintaining ~10x higher throughput than Raft/Multi-Paxos under such circumstances.

## Performance Limitations

All consensus algorithms require round trips. QuePaxa takes 1–3 round trips between the proposer and a majority to decide a proposal (1 if the leader proposes, 3 if a non-leader proposes, more if multiple replicas propose simultaneously). Latency is proportional to the distance between replicas.

Mitigations include:

1. Developers can place replicas closer together.
2. Writes can be batched—10 writes in 10ms can be bundled into one proposal.
3. Stale reads (never inconsistent) are available from any replica's local data without consensus rounds.
4. Multiple operations can be bundled (e.g., compare-and-swap or general transactions).

These limitations make Meerkat well-suited "for control plane information that is written infrequently but must remain consistent."

## Current Status

Meerkat is not yet in production. The team has run multiple proofs-of-concept with up to 50 globally distributed replicas. The key result: leaders in these clusters constantly fail, yet "the cluster keeps operating with no increase in error-rate."

## Future Plans

The team plans to publish blog posts covering:

- How QuePaxa works in detail
- Formal verification of parts of their Rust implementation
- Bootstrapping and cluster management
- Optimal replica placement
- Deterministic simulation testing for bug discovery
- A manuscript for peer review

Tags: Research, Network, Database, Distributed Systems, Meerkat
