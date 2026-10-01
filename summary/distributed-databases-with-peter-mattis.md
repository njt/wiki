---
url: https://www.youtube.com/watch?v=0GzwuYGvKA4
title: "Distributed databases with Peter Mattis"
author: Gergely Orosz (host), Peter Mattis (guest)
date_fetched: 2026-10-02
date_published: 2026
topics:
  - distributed-systems
  - agent-coding-workflow
---

Peter Mattis — GIMP and GTK co-creator, Gmail storage engineer, founding team of Google's Colossus, and co-founder/CTO of Cockroach Labs — talks to The Pragmatic Engineer for a 100-minute career-and-systems interview.

On Google: he joined in 2002 to build Gmail's backend message storage and indexing (Caribou), including threading backed by B-trees and an inverted index. He seeded the build system that became Blaze/Bazel ("make is assembly language for dependencies"), then joined the founding team of Colossus, GFS's successor: a flat-namespace distributed file system whose big win was Reed–Solomon erasure coding (2x storage overhead instead of 3x triplication, with better redundancy), and whose metadata lives in a bootstrap-tangled relationship with BigTable — Colossus stores metadata in BigTable while BigTable runs on Colossus, broken only by a foundational BigTable instance that didn't sit on Colossus.

On CockroachDB: he distinguishes distributed storage systems (large, immutable, append-only files) from distributed databases (small, typed, mutable rows), explaining why LSM trees — LevelDB → RocksDB → Cockroach's own Pebble — fall directly out of Colossus's append-only files. Sharding is range-based partitioning of one contiguous key space with an index on top, which "if you squint is a B-tree"; consensus needs at least three replicas (CockroachDB writes to all three, reads from one, and only pays the consensus read on recovery); strong consistency is implemented not by avoiding writing to all replicas but by doing it fast. Latency across regions is bounded by the speed of light in fibre — hence his aside that the fastest global data path is straight up through Starlink.

On AI coding: he stopped writing code between 2022 and 2024 as CTO, then returned to the keyboard because "to guide people about how to use it, you have to be a user yourself." A Thanksgiving session with Opus materialised a long-postponed CockroachDB test-matrix tool in four days, and he now ships production database code — including a B-tree in 30 minutes ("probably 10,000 lines of highly optimized Rust"). His diagnosis of model weaknesses: agents are lazy about testing and don't scrutinise performance, and the fix is a firm hand — property-based, metamorphic, and deterministic simulation testing demanded explicitly. He expects humans to stop reviewing code the way we stopped reviewing assembly, with guardrails (e.g. an agent that checks decompiled output for zero-overhead abstractions) replacing line-by-line review. Domain expertise is the amplifier: ask a model to build CockroachDB and you get something "ultimately hollow inside"; ask as an expert and you can get something magical.
