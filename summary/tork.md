---
url: https://github.com/runabol/tork
title: "Tork"
author: Arik Cohen (runabol)
date_fetched: 2026-09-15
date_published: 2023
topics:
  - agent-orchestration
  - distributed-systems
---

# Tork

Tork is a general-purpose workflow engine in Go (~15K lines, MIT, v0.1.160 in this clone; README copyright 2023-present) where a job is an ordered list of tasks and each task runs in its own container — Docker, Podman, or a plain shell process. One binary runs in three modes: standalone, coordinator, or worker, so going from laptop to cluster is a flag plus a message broker rather than a new deployment model.

The architecture is deliberately un-fancy: stateless, leaderless coordinators consume tasks from a fixed set of well-known queues (`pending`, `started`, `completed`, `error`, `heartbeat`, `jobs`, `logs`, `progress`, `redeliveries`); workers subscribe to user-defined work queues and run containers. All state lives in a datastore (Postgres in production, in-memory for tests); all coordination happens by passing task structs over the broker. There is no scheduler assigning work and no leader election — correctness comes from transactional, guarded state updates in the datastore.

The task model covers composite execution with one struct: `parallel` fans out child tasks, `each` maps over a list with bounded concurrency (implemented as a database-backed semaphore — each child completion promotes the next `CREATED` child inside the completion transaction), and `subjob` spawns a whole child job, attached or fire-and-forget. Tasks carry `if` conditions, retries, per-task CPU/memory limits, timeouts, priorities (mapped to RabbitMQ's `x-max-priority`), mounts, sidecars, and pre/post tasks that share the parent's mounts and network.

Two implementation details stand out. First, jobs are materialized progressively: the coordinator creates task N+1 only after N completes, evaluating `{{ expr }}` templates (the expr language) against the accumulated job context, so a 1,000-task job has one task row until it starts. Second, task output is captured through a hidden `/tork` volume: the run script is written into the container as an executable `/tork/entrypoint`, and the worker reads results back from `/tork/stdout` and `/tork/progress` files via Docker's copy-from-container API — stdout is streamed to logs in parallel, while the structured result travels through the filesystem, not the log stream.

Crash recovery is broker-native: an unacked task from a dead worker is redelivered, and a coordinator-side handler re-publishes it (failing it after 5 redeliveries). Scheduled jobs use cron with a distributed lock (in-memory or Postgres) that enforces a minimum 10-second hold so clock-drifting coordinators can't instantly re-acquire. Secrets are AES-GCM encrypted at rest and redacted from logs by default. Extension points are middleware chains (web/job/task/node/log) plus registered brokers, datastores, runtimes, and mounters; the README's own use cases — image resizing, video transcoding, CI with Kaniko — position Tork as a self-hosted, container-shaped alternative to workflow engines like Airflow or Temporal at a fraction of their surface area.
