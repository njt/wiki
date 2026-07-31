---
url: https://www.canva.dev/blog/engineering/session-revocations-at-scale/
title: Session revocations at scale
author: Llew Vallis
date_fetched: 2026-08-01
date_published: 2026-07-22
site: Canva Engineering Blog
---

# Session revocations at scale

How Canva keeps hundreds of millions of user sessions fast and secure

Llew Vallis, Canva Engineering Blog, Jul 22, 2026

---

Canva manages sessions for hundreds of millions of users, with every backend request needing to identify the logged-in user — answered hundreds of thousands of times per second. Canva uses encrypted browser cookies containing user identity, permissions, and roles, allowing gateways to trust cookie details without a networked datastore lookup on every request. When users are logged out or permissions change, revocations must propagate in near real time.

## Pre-existing Architecture

- Each gateway maintains an in-memory lookup of revoked sessions for speed and reliability (rather than checking a networked datastore per request).
- 12 hours of revocations are stored in memory; slower MySQL lookups are used during periodic session cookie refreshes.
- On startup, hundreds of gateway pods each pulled over a million revocations from MySQL, causing a "coordinated stampede" on the database during deployments. Mitigations were temporary (adding read replicas).

## Optimizing the In-Memory Cache

- **Redis evaluated and rejected:** Not typically deployed in a fully durable configuration, and would require managing a cluster plus consistency complexity.
- **S3 chosen** because it "is designed to serve downloads of large, durably stored files at a low cost" and offers strong durability.
- **Sliding window partitioned into 30-minute segments**, each stored as one S3 object. Chunk names encode the start timestamp, enabling efficient sorted-key lookups after a cutoff.
- **Binary format:** Each revocation packs a principal (who it applies to) and a login timestamp cutoff into 16 bytes via bit twiddling. Chunks are flat, sorted arrays of these 16-byte elements, enabling binary search per principal. Gateways operate directly on downloaded bytes with no transformation.
- **Memory footprint reduced by 8×** (87.5%) versus the prior multi-Java-object representation.
- **Multiple revocation types supported** by reserving flag bits (e.g., invalidating cached cookie info without full logout, or targeting an entire brand), while still sorting by principal.

## Keeping Chunks Up to Date

- An asynchronous worker process scans the database, fetches large un-uploaded batches, reads the latest chunk (or creates a new one if the window has advanced), inserts revocations into the sorted array, and re-uploads.
- **Optimistic concurrency control via conditional PUT requests** — preconditions assert the chunk hasn't changed since read; new-chunk creation uses similar checks. "The PUT conditions guarantee that each read-modify-write operation only ever appends to the set of revocations in S3."
- **ZooKeeper leader election** reduces conflicts/load, but correctness doesn't depend on it (a paused node could otherwise overwrite a new leader's writes).
- **Scaling note:** Building a chunk of N revocations is ~O(N²), but with hundreds of revocations per batch, throughput exceeds 2,000 revocations per second — more than needed for the foreseeable future.

## Downloading the Chunks

- Gateways download only chunks from the most recent 12 hours (all tokens refresh within this window).
- Conditional GET requests redownload the latest chunks only when changed.
- Even 1 million revocations in a 30-minute window equals only 16 MB, polled a few times per minute — negligible versus gateway proxy traffic.
- Chunks older than 12 hours are dropped from cache.
- Per gateway: tens of megabytes; across the fleet: tens of gigabytes.

## Outcome and Learnings

- Improved deployment speed; reduced session revocation database read replicas to two (for redundancy).
- Database load now scales predictably with write throughput and site traffic, rather than with prior writes × gateway instances downloading the cache.
- Key takeaway: scaling in theory vs. practice — the worker is actually bottlenecked on network latency, not on sorting dense arrays.
- The team implemented and tested multiple distributed system designs at expected scale on real infrastructure, then used end-to-end tests and human review for the production version.

## Acknowledgments

Thanks to Joeby Neil (collaborator), Martin Doms (support), plus Dennis Kao, Michael Yates, Zac Sims, Stu Liston, Min Coombes, and Simon Newton for feedback.
