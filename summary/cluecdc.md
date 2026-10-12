---
url: https://github.com/cluedata/cluecdc
title: "ClueCDC"
author: ClueData (cluedata)
date_fetched: 2026-10-13
date_published: 2026 (open-source release, Apache-2.0)
topics:
  - databases-and-data
  - developer-tools
---

ClueCDC is an open-source (Apache-2.0) **control plane** for building, managing and monitoring change-data-capture pipelines built on Debezium, Apache Kafka and Kafka Connect. Postgres 13+ and MySQL 8 sources are supported today, with destination adapters including Postgres and object storage. The core architectural claim is stated in the README and honoured in the code: **CDC records never flow through the ClueCDC API** — the control plane only configures and observes the data plane (Debezium connectors, Kafka, delivery connectors), all of which run in Kafka Connect.

On top of that separation it offers a five-step UI wizard (Next.js web app) that validates and deploys a capture plus its first delivery, connection testing, schema/table discovery with "CDC readiness" checks (primary keys, replica identity, binlog settings), explicit table selection, snapshot policies (initial / streaming-only / never / always), per-table add/remove/resnapshot/stop operations, and pause/resume/restart/retry pipeline lifecycle. Metadata, audit history, alerts and metrics live in its own Postgres-backed store.

The backend is a Python FastAPI service (`apps/api/app/main.py`, ~30 modules: services/, adapters/, providers/, alerts/, workers/) with a Next.js/React frontend (`apps/web`, ~98 TS/TSX files). A background worker (`app/workers/worker.py`) claims jobs from a Postgres-backed job table with lease-based claiming, expiry recovery and attempt caps. Secrets are Fernet-encrypted in the metadata database and handed to Kafka Connect via a `${cluecdc:<uuid>:password}` ConfigProvider reference resolved over an internal, token-authenticated `/internal/secrets` endpoint — passwords never appear in connector configs.
