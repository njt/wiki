---
url: https://celld.dev/
title: celld
author: Deno Land Inc.
date_fetched: 2026-08-14
topics:
  - databases-and-data
---

# celld.dev — Summary

The marketing site for celld, Deno's self-hosted, distributed Durable Objects
runtime. Where the GitHub README ([[raw/celld]]) documents the architecture, this
page makes the *argument*: that you can keep Cloudflare's Durable Objects
programming model while reclaiming placement, state, and operational evidence.

The headline mechanism is compressed to a single line: "The bucket is the
coordinator — no membership protocol, no failure detector, no consensus.
Ownership is a record in your bucket, claimed with one atomic write." celld's
built-in replicator continuously ships each cell's SQLite state to the bucket as
LTX segments.

The pitch has three beats. First, a cell's identity isn't fused to a machine —
ownership is a lease granted by compare-and-swap, so a lost node is replaced by
another acquiring the lease and restoring the cell in seconds. Second, tenancy:
with no shared scheduler or placement layer, your workload can't be coupled to
another customer's. Third, operational evidence: when a cell misbehaves, the
ownership record, SQLite and LTX files, and logs sit on your disk, so you answer
"what happened to my cell" with `sqlite3` and `grep`, not a status page.

The page is honest about what does *not* change: "Your fleet still depends on its
machines, network, and bucket provider." Running it yourself moves the failure
surface rather than eliminating it. Install is `curl -fsSL celld.dev/install.sh | sh`
or `docker run ghcr.io/denoland/celld`; the site's demo prompt is "create a
distributed chat app with vite, use celld.dev and exe.dev, 2 VMs".
