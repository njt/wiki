# ClueCDC

ClueCDC is an open-source control plane that wraps Debezium + Kafka + Kafka Connect into an operable product: one UI and API for connections, table discovery, connector lifecycle, per-table resnapshots, health, alerts and audit. Its defining discipline is that it is purely a **control plane** — CDC records flow Debezium → Kafka → delivery connectors and never transit the ClueCDC API. That sounds like a restatement of Kafka Connect's own design, but the project's real work is everything Connect leaves out: readiness validation, per-table orchestration, secret handling, error classification, and state reconciliation between an authoritative config and the messy reality of a Connect cluster.

---

## Architecture

A TypeScript/Python monorepo:

- **`apps/api`** — FastAPI + SQLAlchemy async + Alembic, the control plane. ~30 modules in a clean layered layout:
  - `app/adapters/` — thin HTTP clients for Kafka Connect (`connect.py`), Kafka (`kafka.py`), and direct database adapters (`postgres.py`, `mysql.py`).
  - `app/providers/` — per-database plugin layer behind a `DatabaseProvider` Protocol (`base.py`) with a `Capability` StrEnum (`CDC_SOURCE`, `SNAPSHOT`, `SCHEMA_EVOLUTION`, `DELETE_PROPAGATION`, …) and a registry. Postgres and MySQL implement it; `docs/development/adding-connector.md` documents this as the extension seam.
  - `app/services/` — domain logic: `pipeline.py` (create/deploy/reconcile, 491 lines), `debezium.py` (config builders), `pipeline_tables.py` (per-table operations incl. incremental snapshot signalling), `snapshots.py`, `destinations/`, `secrets.py`.
  - `app/workers/worker.py` — background job runner.
  - `app/alerts/` — event bus, error classifier, rules, and Slack/Telegram/webhook dispatchers.
- **`apps/web`** — Next.js console (~98 TS/TSX files) implementing the five-step pipeline wizard.
- **`packages/contracts`** — shared TypeScript types between web and API.
- **Data plane** — Debezium source connector and delivery connector run inside Kafka Connect workers; Kafka in the middle. `deploy/` ships both Docker Compose and Kubernetes manifests with independently scalable Connect workers.

The metadata model (`app/models/entities.py`, ~27 SQLAlchemy entities) is telling: `Pipeline`, `PipelineTable`, `PipelineOperation`, `PipelineEvent`, `PipelineMetric`, `AuditLog`, `Job`, `AlertRule`, `NotificationChannel` — the control plane keeps a full desired-state and event-history ledger, not just connector configs.

## Key techniques

- **Desired-state reconcile loop.** `services/pipeline.py` re-derives the actual connector configuration from pipeline metadata (`input_for` → `derive_actual_state`) rather than trusting what's in Connect, then reconciles. The API models Connect failures as `DomainError`s with upstream status codes (`connect.py` maps 409 to "rebalancing or already exists").
- **Secrets never in connector configs.** Connector config carries only `${cluecdc:<uuid>:password}`; Kafka Connect's ConfigProvider resolves it by calling the control plane's internal `/internal/secrets/{reference}` endpoint (main.py), guarded by a constant-time `secrets.compare_digest` bearer check and an allowlist of key names. Secrets are Fernet-encrypted at rest (`services/secrets.py`), with a `SecretProvider` Protocol seam for external vaults. All Connect API responses pass through a `redact()` helper, and task `trace` blobs (which contain credentials) are stripped and replaced with a `failure_hint()` classification before any client sees them.
- **Per-pipeline Debezium isolation.** `debezium.py` derives slot/publication/signal-table names from `sha256(topic_prefix)[:20]`, so pipelines don't collide on Postgres replication slots; `slot.drop.on.stop=false` with documented DBA cleanup — an honest trade-off of durability over tidiness.
- **MySQL server-ID allocation.** `providers/server_ids.py` deterministically maps pipeline UUIDs into a 540M-range server-id space and resolves collisions by linear probing against the metadata DB — a small but real distributed-systems problem most Debezium users hit manually.
- **Incremental snapshots via Debezium signals.** Per-table resnapshot works by creating a per-pipeline signal table in the source database, adding it to the Postgres publication, and writing a signal row (`pipeline_tables.py`); the control plane then tracks snapshot progress per `PipelineTable`.
- **Postgres-as-queue jobs with leases.** `workers/worker.py` claims jobs with `UPDATE ... WHERE status='PENDING'`, and reclaims expired leases (requeueing or failing by attempt count) in the same claim pass — a classic lease-based scheduler with no external broker.
- **Error classification as data.** `alerts/classifier.py` is a rule table mapping raw infra error text ("requested wal segment", "binlog purged", "replication privilege") to stable event types like `CDC_WAL_ERROR`, `CDC_PERMISSION_ERROR`; `adapters/connect.py` `failure_hint()` does the same for sink tasks. Operators see a human diagnosis, not a stack trace.

## Design decisions

- **Control/data separation, strictly.** The payoff is that the control plane can be down, restarted or replaced without touching data movement; the cost is that ClueCDC must maintain a mirror of connector state and constantly reconcile it (hence `derive_actual_state` and the whole reconcile layer).
- **Capability-driven provider registry** over if-else on database type. Adding a connector means implementing one Protocol; the UI discovers capabilities from `ProviderMetadata`.
- **A healthy capture path stays available when a sink degrades**: `aggregate_pipeline_state` reports `DEGRADED`, not `FAILED`, when only delivery is broken — a deliberate operational semantic.
- **Encrypted-in-DB secrets rather than mandatory vault integration** — pragmatic for self-hosted open source, with the provider seam left for KMS-backed deployments.
- **Weakness**: the error classifier is string matching over lowercased messages, brittle across Debezium/JDBC version churn; the codebase acknowledges this by keeping rules as an easily edited table and falling back to "inspect worker logs".

## Comparison notes

- [[Streambed]] goes the opposite direction: a single Go binary doing Postgres→Iceberg CDC directly against the WAL with Parquet+S3, explicitly *without* Kafka/JVM. ClueCDC keeps the full Kafka Connect stack and instead sells operability — multi-source, multi-destination, UI, alerts — which Streambed deliberately sheds.
- [[Artie]] is the managed-SaaS analogue: sub-minute replication with zero data retention and no Kafka. ClueCDC is the self-hosted, Kafka-native answer to the same "Debezium is too raw to operate" pain.
- [[Postgres to Snowflake Data Mirroring]] pushes CDC into the database itself via an extension; ClueCDC keeps extraction external in Debezium, betting on broad connector support over tight source integration.
- The architecture vindicates [[The Log — Unifying Abstraction for Real-Time Data]]: ClueCDC exists precisely because the log (Kafka) and its capture machinery are excellent but operationally bare — it adds the control plane Kreps' abstraction never specified.

---
*Sources: [[raw/cluecdc]], [[summary/cluecdc]]*
*Last updated: 2026-10-13*
