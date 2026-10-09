---
url: https://tlockney.github.io/celld-book/
title: "celld: Durable Objects on Your Own Storage"
author: Thomas Lockney (curator; book generated with AI from curated materials)
date_fetched: 2026-10-09
date_published: 2026-09-26
topics:
  - distributed-systems
  - coding-agents-and-frameworks
---

A book-length technical reference for celld v0.6.0, Deno's self-hosted runtime for the Cloudflare Durable Objects programming model. celld keeps the model — a named, single-threaded object with its own SQLite database, addressed by name — but replaces Cloudflare's placement and storage layer with the reader's own machines and one S3/GCS/Azure object-storage bucket as the sole coordinator. There is no membership protocol, no failure detector, and no consensus service: ownership of a cell is a record in the bucket claimed with one atomic conditional write, and every activation advances an epoch used for fencing.

The durability story is explicit and size-dependent. With one node every write waits for the bucket (bucket proof); with two or more nodes the owner replicates to one or two followers and acknowledges once they have the data on disk (fleet proof), with the bucket upload following. RPO=0 is earned, not assumed: the output gate holds every response until a durability proof covers it, and a fenced node discovers it is stale by re-reading its ownership record, not by comparing clocks. The bucket must provide conditional create, conditional overwrite, read-after-write consistency, and exact ranged reads — a table of fleet-qualified stores includes S3, R2, GCS, Tigris, and Azure Blob, and excludes B2, Hetzner, and DigitalOcean Spaces.

The book is also a survey: Part I gives the best compact treatment of the actor model lineage (Hewitt 1973, Erlang's let-it-crash, Akka, Orleans' virtual actors) and of durable execution (replay, determinism, idempotent steps, five engines from Temporal to DBOS) that explains where Durable Objects sit. Application guidance follows: model the thing that persists as a cell, the thing that finishes as a Workflow; keep constructors light because they run on every hibernation wake; make operations idempotent because celld never retries after transmission begins. The compatibility chapter shows the Cloudflare surface celld reimplements (D1, KV, Queues, Workflows, R2, Dynamic Workers, facets, containers) and where the line is drawn. Labs are executed notebooks against a live node, treated as evidence rather than illustration. The colophon states the book was AI-generated from materials Lockney curated — celld's docs and release notes, Cloudflare's posts, and the actor/durable-execution literature.
