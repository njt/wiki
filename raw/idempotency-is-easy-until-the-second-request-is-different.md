---
url: https://blog.dochia.dev/blog/idempotency/
title: "Idempotency Is Easy Until the Second Request Is Different"
author: Dochia CLI Blog
date_fetched: 2026-05-15
date_published: 2026-05
note: Original URL returned HTTP 403. Content reconstructed from web search results (Google, Pinboard) and HN discussion (item 48047930). Wayback Machine had a capture dated 2026-05-10.
---

# Idempotency Is Easy Until the Second Request Is Different

~25 minute read. Argues that idempotency is much harder than most developers think — the happy-path replay cache is table stakes, and the real engineering is in the edge cases.

## 1. The Hard Cases Beyond Simple Replay

The canonical idempotency pattern (store request key → return cached response on retry) breaks down in these scenarios:

- **Completed replay** — stored response may be stale or contain PII the caller shouldn't get on replay
- **Concurrent retry** — second request arrives while the first is still processing
- **Partial local success** — crash after local DB commit but before publishing event
- **Downstream unknown state** — provider timeout/crash after accepting the request
- **Same key, different command** — first request was $10, retry sends $100 with same key
- **Duplicate operation without a key** — operation has no natural idempotency key
- **Retry after expiry** — idempotency window has lapsed
- **Retry after deploy** — code changed, replaying stored response may not make sense
- **Retry after schema change** — stored response body no longer matches current schema
- **Retry after region failover** — idempotency store in old region is unreachable

## 2. Same Key + Different Content = 409 Conflict

A scoped idempotency key reused with a *different canonical command* should be a **hard error**, not silent replay:

```
HTTP/1.1 409 Conflict
{ "errorCode": "IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_REQUEST" }
```

This prevents the caller from accidentally retrying a modified payload under the same key and getting the *old* result back silently.

## 3. Hash the Command, Not the Raw Bytes

To detect "same command" vs "different command":

- Parse into a versioned DTO; normalize values (amounts, enum casing, defaults, timestamp precision)
- Exclude transport-only metadata, `Authorization` header, and the idempotency key itself
- Include path parameters, operation name, and semantic headers (e.g., API version)
- Use canonical serialization and a stable hashing algorithm

## 4. Atomic Ownership via "Insert-First" Pattern

```sql
INSERT INTO idempotency_requests (...)
VALUES (...) ON CONFLICT DO NOTHING;
```

- If `rows_inserted == 1` → this request owns execution
- Otherwise, load the existing row and branch on status (`COMPLETED`, `IN_PROGRESS`, `UNKNOWN_REQUIRES_RECOVERY`)

## 5. State Machine for External Side Effects

When a downstream provider call is involved, the local state machine transitions:

```
RECEIVED → LOCAL_PAYMENT_CREATED → PROVIDER_REQUEST_SENT → PROVIDER_CONFIRMED → COMPLETED
```

A crash after `PROVIDER_REQUEST_SENT` requires a **recovery worker** that queries the provider by a stable downstream key — *not* blindly retrying.

## 6. Idempotency Table Schema (PostgreSQL)

```sql
CREATE TABLE idempotency_requests (
    tenant_id       TEXT NOT NULL,
    operation_name  TEXT NOT NULL,
    idempotency_key TEXT NOT NULL,
    request_hash    TEXT NOT NULL,
    status          TEXT NOT NULL,
    response_status INT,
    response_body   JSONB,
    resource_type   TEXT,
    resource_id     TEXT,
    error_code      TEXT,
    created_at      TIMESTAMPTZ NOT NULL,
    updated_at      TIMESTAMPTZ NOT NULL,
    expires_at      TIMESTAMPTZ NOT NULL,
    locked_until    TIMESTAMPTZ,
    PRIMARY KEY (tenant_id, operation_name, idempotency_key)
);
```

## 7. Redis Is Not Enough

`SET NX EX` can reduce duplicate concurrent execution but is *not* durable memory of the operation outcome. If the lock expires mid-call or the process crashes after provider success, the retry has no way to know what happened.

## 8. Replay Is a Contract

Returning an `Idempotent-Replayed: true` header helps debugging. The response body can either be a faithful stored replay or reconstructed from a resource reference — both approaches have trade-offs (PII retention vs. stale representation).
