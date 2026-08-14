---
url: https://celld.dev/
date_fetched: 2026-08-14
---

`$ curl -fsSL celld.dev/install.sh | sh``$ docker run ghcr.io/denoland/celld``❯ create a distributed chat app with vite, use celld.dev and exe.dev, 2 VMs`The bucket is the coordinator — no membership protocol, no failure detector, no consensus. Ownership is a record in your bucket, claimed with one atomic write. celld's built-in replicator continuously ships each cell's SQLite state to that bucket as LTX segments.

Durable Objects is a strong programming model. celld keeps that model while moving placement, state, and operational evidence into infrastructure you choose.

**What changes when you run the model yourself.**

A cell's identity isn't fused to a machine — ownership is a lease in your bucket, granted by compare-and-swap. Lose a node and another acquires the lease and restores the cell in seconds: your fleet reading your storage, not a vendor restoring a placement you can't see.

Your fleet still depends on its machines, network, and bucket provider. What changes is tenancy: no shared Durable Objects scheduler or placement layer can couple your application to another customer's workload.

When a cell misbehaves the evidence is on your disk — the ownership record, the SQLite and LTX files, and the logs. You answer “what happened to my cell” with sqlite3 and grep, not a status page that declines to say.
