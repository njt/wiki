---
url: https://github.com/TerseAI/durable-actors
title: "Durable Actors — an open-source alternative to Cloudflare Durable Objects"
author: TerseAI
date_fetched: 2026-10-09
date_published: 2026-10-09
topics:
  - distributed-systems
  - developer-tools
---

Terse's Durable Actors is an open-source, self-hostable alternative to Cloudflare Durable Objects: stateful serverless actors with durable state and serialized execution, aimed at real-time apps — chat, collaborative documents, and agent swarms. The pitch is no vendor lock-in, no memory limits, and built-in observability, with a live demo, TypeScript and Python SDKs, one-command local dev, and self-hosting on GCP.

The repo is a Rust workspace (~25K lines) implementing a regional control plane, sandbox "actor hosts", and the durability runtime. Actors are classes whose `@Persisted` fields survive restarts; `@Interleave` opts a method into reentrancy so other calls run during an await (e.g. new socket connections while an LLM reply streams). `durable-actors generate` compiles the actor definitions into type-safe TS/Python clients; the frontend connects over a WebSocket grant issued by your backend.

Under the hood: each actor's state is a SQLite database snapshotted and replicated via an embedded Litestream fork (terse-ltx/terse-litestream) to a GCS bucket, with an optional "Rapid" log-structured segment store for lower state-write latency; a Postgres-backed control plane owns placement leases (`owner_epoch` fencing), a GKE Sandbox (gVisor) spare-pool for fast host provisioning, and a gRPC/JSON control-plane protocol. Observability is built in as ordered latency milestones — deliberately not distributed traces — across six event families.
