---
url: https://blog.verygoodsoftwarenotvirus.dev/posts/2026/09/23/why-didnt-anybody-tell-me-about-hash-slots
title: "Why didn't anybody tell me about hash slots"
author: "Eric Stibbens (verygoodsoftwarenotvirus)"
date_fetched: 2026-09-25
date_published: 2026-09-23
topics:
  - databases-and-data
  - software-engineering-craft
---

A delivery-app engineer describes speeding up a courier-matching service that leans heavily on a routing engine, by building a shared Redis cache of route estimates keyed on H3 hex IDs. The naive `<origin hex>:<dest hex>:<resolution>` key layout collides with a Redis cluster's 16,384 hash slots: multi-key commands like `MGET`/`MSET` must land in one slot, so every key with a unique hex pair scatters across nodes and the client silently shards one read into scores of single-key round trips.

The fixes come one discovery at a time: Redis **hash tags** (`{...}` braces override which part of the key is hashed) let the team brute-force integer tags — Bitcoin-mining style — until every cluster node owns enough of them; keys are grouped by slot locally before fan-out so each node gets one batched request instead of a queue; `MSET`'s lack of per-key expiry is solved with an `EVAL` script instead of pipelined `SET ... EX` or a crash-window-prone `MSET`+`EXPIRE` pair; and swapping JSON values for CSV reclaimed surprising CPU at scale.

The coda generalizes past Redis: the author kept invoking Rob Pike's rules — "Measure. Don't tune for speed until you've measured" and "fancy algorithms are slow when n is small" — but realised his `n` was reliably large, and that he should have built the benchmarking harness *first*. In the post-Claude era, when a harness is "a matter of providing a spec and waiting 10 minutes," skipping one to protect a ticket estimate is no longer acceptable.
