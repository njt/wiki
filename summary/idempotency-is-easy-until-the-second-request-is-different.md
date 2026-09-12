---
url: https://blog.dochia.dev/blog/idempotency/
title: "Idempotency Is Easy Until the Second Request Is Different"
author: Dochia
date_fetched: 2026-05-18
date_published: 2026-05-07
topics:
  - software-engineering-craft
---

# Idempotency Is Easy Until the Second Request Is Different

25 min read. Tags: api, http, idempotency, backend, distributed-systems, databases, microservices, architecture, payments.

## Core Argument

People talk about idempotency like it's a solved problem: put an `Idempotency-Key` on the request, store the response, replay it on retry. That version survives the demo. The hard part starts with the *second request*, because the second request is not always a clean replay of the first.

The cases that matter are the ones a replay cache does not explain:

- Completed replay
- Concurrent retry
- Partial local success
- Downstream unknown state
- Same key with a different canonical command
- Duplicate operation without a key
- Retry after expiry
- Retry after deploy, schema change, service hop, or region failover

**Thesis:** If your design only handles completed same-command retries, it is a replay cache. That might be enough for some endpoints, but it is not the whole problem.

## Idempotency Is About the Effect

An operation is idempotent if applying it once or many times has the same intended *effect*. The word doing all the work is "effect."

HTTP gives method-level semantics (PUT idempotent, DELETE idempotent, POST usually not), but your handler can still produce repeated side effects the business cares about: duplicate audit records, domain events, emails, provider calls, or metrics that affect billing or fraud logic.

A uniqueness constraint can prevent one class of duplicate. It does not, by itself, give the client a correct retry result. `unique(account_id, merchant_reference)` might prevent two payment rows, but if the retry gets a generic 500, the client still doesn't know whether the payment succeeded.

## What You Need to Remember

For `POST /payments`, the durable idempotency record needs to answer three questions:

1. Who owns this key?
2. What did the first command mean?
3. What outcome can be replayed?

### Schema

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

The key is not globally unique unless you deliberately make it global. Usually it should not be. Scope might be tenant, user, account, merchant, API client, or some combination.

The `operation_name` prevents accidental reuse across different operations. A key used for `create_payment` should not automatically mean the same thing for `create_refund`.

`request_hash` is the server's memory of the first command. Without it, same key + different body becomes ambiguous.

### Status Behavior Table

| Existing record | Same canonical command? | Suggested behavior |
|---|---|---|
| none | yes | insert IN_PROGRESS and execute |
| COMPLETED | yes | replay stored response or documented equivalent |
| any existing record | no | reject with idempotency conflict |
| IN_PROGRESS, fresh | yes | wait, return 202, or return 409 + Retry-After |
| IN_PROGRESS, stale | yes | recover ownership; do not blindly execute again |
| FAILED_REPLAYABLE | yes | replay stored failure |
| FAILED_RETRYABLE | yes | allow retry according to policy |
| UNKNOWN_REQUIRES_RECOVERY | yes | trigger reconciliation or return pending/recovery status |
| expired/deleted | unknown | follow documented expiry behavior |

## Same Key, Different Command

This is the bug the idempotency layer should catch loudly.

First request: `amount: "10.00"`. Second request: `amount: "100.00"`. Same `Idempotency-Key: abc-123`.

Returning the original response hides a serious client bug. The client asked for a 100 EUR payment and got back a 10 EUR payment. If the caller does not compare the response carefully, it may believe the 100 EUR payment succeeded. That is not idempotency. That is reinterpretation.

**Recommendation:** For side-effecting APIs, a scoped key reused with a different canonical command should be a hard error, regardless of whether the first operation completed, failed, or is still running.

```
HTTP/1.1 409 Conflict
Content-Type: application/json
{
    "errorCode": "IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_REQUEST",
    "message": "This idempotency key was already used with a different request."
}
```

409 Conflict is a defensible default because the request conflicts with the server's remembered meaning for that scoped key. Some APIs use 400 or 422; the important part is a stable machine-readable error and no silent replay for a different command.

A common client bug: `idempotencyKey = cartId` instead of `idempotencyKey = paymentAttemptId`. The server should not guess which payment the cart key was supposed to represent.

## Hash the Command, Not the Bytes

Raw byte comparison is usually too strict for JSON APIs. Field order and whitespace should not matter.

Edge cases that need decisions:
- **Defaults:** `{"channel": "web"}` vs `{}` when `channel: "web"` is the server default — are these the same?
- **Unknown fields:** If your API ignores unknown JSON fields, do `{"foo": "bar"}` and `{}` hash the same? If fields might become meaningful after a deploy, perhaps no.

**Practical rule:** Hash the validated command, not the raw HTTP body.

1. Parse the request into a versioned request DTO or command
2. Normalize values your API treats as equivalent: amounts, enum casing, default fields, timestamp precision
3. Exclude transport-only metadata
4. Include path parameters and operation name
5. Include semantic headers if they affect the operation (e.g., API version)
6. Exclude `Authorization` and the idempotency key itself
7. Serialize canonically
8. Hash with a stable algorithm

Be careful with amounts, timestamps, generated defaults, locale-sensitive formatting, and fields added during deploys. The request hash is a contract. If you change how it's computed, old retries can start looking different.

## Atomic Ownership: Insert-First

Two identical requests hitting two API instances at nearly the same time. Both observe no existing row. Both execute the side effect.

The fix is insert-first, not check-then-insert:

```sql
INSERT INTO idempotency_requests (...)
VALUES (...) ON CONFLICT DO NOTHING;
```

If `rows_inserted == 1`: this request owns execution. Otherwise, load the existing row and branch on status.

Recovery ownership must be acquired atomically too. Otherwise two retries can both decide the old owner is dead and both start recovery.

## Local vs. External Side Effects

The nice version: one database transaction covers the idempotency row, the business row, and the outbox event.

External side effects change the shape. Holding a database transaction open while calling a provider is usually a bad idea. Committing before the provider call means local state says IN_PROGRESS while execution continues outside the transaction. If the process crashes there, a retry has to recover.

### Redis SET NX EX

Often proposed as the whole solution. At best, it's an execution guard. It is not durable memory of the operation outcome. If the Redis lock expires while the provider call is still running, another request can enter. If the process dies after the provider succeeds but before storing the response, the lock does not help the retry know what happened.

Redis can be useful. It is not a substitute for remembering the operation outcome.

### The Provider Timeout

The failure path that matters:

1. API receives POST /payments
2. Inserts idempotency row as IN_PROGRESS
3. Creates local payment `pay_789`
4. Calls downstream payment provider
5. Provider receives the request and succeeds
6. API times out, crashes, or loses the provider response
7. Client retries with the same key

If the provider received your request and your process died before recording the result, your database cannot infer whether money moved.

**Recovery pattern:** Use a stable downstream operation ID. A recovery worker or retrying request can:
- Acquire recovery ownership for `pay_789`
- Query the provider by `provider_payment_pay_789` (if the provider supports it)
- If confirmed, mark the operation COMPLETED and replay the response
- If the provider cannot answer, mark `UNKNOWN_REQUIRES_RECOVERY`

If the provider has no idempotency key and no query API, your system has an operational gap.

### Internal Status Set

```
IN_PROGRESS
COMPLETED
FAILED_REPLAYABLE
FAILED_RETRYABLE
UNKNOWN_REQUIRES_RECOVERY
EXPIRED
```

Don't expose every internal state directly. But internally, pretending every failure is either "done" or "not done" makes recovery harder.

## Replay Is a Contract, Not a Convenience

For a completed idempotent request, replaying the same status and body is the least surprising behavior. Use `Idempotent-Replayed: true` header for debugging (don't make clients depend on it).

**Stored response vs. reconstructed response:**

Storing full responses gives faithful replay but can retain PII, signed URLs, one-time tokens, or cardholder data. Reconstructing from a resource reference saves space but can return a different representation if the resource changed after creation.

Schema changes make this worse. If a generated client retries after a deploy, should it receive the stored v2 response or a reconstructed v3 response? Both can be defensible. They are different contracts.

A common compromise: store `resource_type`, `resource_id`, `response_status`, `response_schema_version`, and store full response bodies only for endpoints where exact replay matters. If you store bodies, treat the idempotency table like sensitive data storage, not like a harmless cache.

## Queue Consumers Have the Same Bug

HTTP gets most of the attention because the header is visible. A lot of duplicate side effects happen later, in consumers, outbox publishers, inbox processors, and notification workers.

A consumer receiving `PaymentCreated(pay_789)` twice should not send two emails, create two ledger entries, or notify a provider twice.

The dedupe key might be the event ID, message ID, operation ID, aggregate ID plus version, or a business key. The right answer depends on the side effect.

**Consumer inbox pattern:**

```sql
CREATE TABLE consumer_inbox (
    consumer_name TEXT NOT NULL,
    message_id    TEXT NOT NULL,
    status        TEXT NOT NULL,
    processed_at  TIMESTAMPTZ,
    error_code    TEXT,
    UNIQUE(consumer_name, message_id)
);
```

But marking the message processed is not trivial:
- Mark processed *before* sending email → crash → retry skips the email forever
- Send email *before* marking processed → crash → retry may send it again

The fix: make the side effect durable before sending it. Insert an email notification row with a unique key, then have a sender process that row.

**Key insight:** Exactly-once delivery is not exactly-once business effect. The latter comes from durable operation IDs, unique constraints, idempotent writes, and recovery paths.

## Expiry Is Part of the API Contract

Idempotency records cannot usually live forever. If the server promises a 24-hour idempotency window, a retry after 25 hours may create a new operation. The replay window is a product/API decision, not just a cleanup setting.

After expiry, you might delete the response body but retain metadata longer (key, scope, operation_name, request_hash, resource_id, timestamps) for diagnostics.

**Stale IN_PROGRESS needs separate handling.** A retry that sees a stale IN_PROGRESS should not blindly execute again. It should acquire recovery ownership, inspect the resource, query downstream if needed, and move the operation to a terminal state.

**Bad cleanup:** `DELETE FROM idempotency_requests WHERE expires_at < now();`
This can delete in-progress records and allow duplicate side effects.

Better: delete in small batches, partition by `expires_at`, drop old time partitions after the replay window, and keep separate retention policies for response bodies and metadata.

## Failure Replay Is a Policy Decision

Pure syntactic validation failures usually don't need idempotency storage — repeating will fail again.

Business rejections are different. If the decision depends on mutable state (balance, inventory, account status, fraud rules), decide whether the first decision is binding for that idempotency key or whether the client must retry with a new key.

**Auth failures** should not create idempotency records. Be careful with authorization failures: a retry must still resolve to the same scope/principal that created the original record.

**Rate limits** should not be recorded as completed idempotent outcomes. A retry later might be allowed.

**Server error before side effects** can often allow retry. **Server error after side effects** is dangerous — if you created the payment but failed to serialize the response, the retry should not create another payment.

## When One Transaction Cannot Cover the Operation

The useful distinction is not monolith vs. microservices. It's whether one durable transaction can cover the operation.

If one database transaction can cover the idempotency row, payment row, and outbox record, the local part is straightforward. When side effects cross boundaries, every boundary that can repeat work needs its own duplicate-suppression rule.

A better model maintains stable operation identities:

```
client idempotency key:  abc-123
payment operation id:    payop_456
payment id:              pay_789
ledger entry id:         ledger_payment_pay_789
email dedupe key:        receipt_payment_pay_789
provider idempotency key: provider_payment_pay_789
```

Each side effect has a durable identity appropriate to that side effect.

**Multi-region:** A region-local idempotency table only protects retries that land in the same region. You either need to route all requests for the same scoped key to a home region, use a strongly consistent shared store, or rely on downstream business constraints.

## When Not to Build a General Idempotency Layer

The cost is not the header. The cost is the durable memory and recovery behavior behind it.

- Don't build payment-grade idempotency for admin actions where duplicates are harmless
- For read-only operations, idempotency keys usually add noise
- If duplicate analytics events cost almost nothing, a heavy idempotency table may be wrong
- Sometimes a business key (`unique(account_id, merchant_reference)`) is better than a random key
- Sometimes change the resource model: `PUT /accounts/acc_1/settings/default-currency` is naturally idempotent

Use the amount of harm from duplicate side effects, the likelihood of retries, and the difficulty of detecting duplicates after the fact to decide how much machinery you need.

## Failure Modes Worth Testing

Tests the author would rather see than a dozen happy-path unit tests:

1. **Same key, same canonical command, completed** — second request returns stored result, does not create `pay_790`, does not publish second event
2. **Same key, different canonical command** — reject with stable machine-readable conflict
3. **Two concurrent identical requests** — one wins execution, side effect executes once. If this passes without a unique constraint or atomic insert, be suspicious of the test
4. **Timeout after downstream success** — retry should not call provider with new operation identity. Should find completed state, query provider, or move into recovery
5. **Duplicate message from a queue** — one ledger entry, one email notification, one provider notification. If first attempt fails halfway, retry completes missing work without duplicating completed work
6. **Expired or stale state** — retry after expiry, retry while stale IN_PROGRESS, retry after response schema changed, retry from another region

## Monitoring

Metrics that find bugs:

- `idempotency.replay.count`
- `idempotency.conflict.different_request.count`
- `idempotency.in_progress.age.max`
- `idempotency.expired_retry.count`
- `idempotency.unknown_state.count`

Replay count is mostly capacity planning. Different-body reuse, stale IN_PROGRESS rows, expired retries, and unknown states are the metrics that find bugs.

## Checklist Before Shipping

- Reject same scoped key + different canonical command
- Use a unique constraint or atomic insert on the scoped key
- Hash the validated command, not raw JSON bytes
- Treat IN_PROGRESS as API-visible behavior
- Define fresh, stale, completed, retryable failure, replayable failure, and unknown states
- Store enough response data to satisfy your replay contract
- Make downstream calls idempotent too, or have reconciliation
- Use outbox/inbox patterns where events and queues are involved
- Do not mark messages processed before their durable side effects exist
- Define the idempotency window as part of the API contract
- Retain metadata separately from sensitive response bodies if needed
- Test concurrent duplicates, timeout after downstream success, partial failure, expiry, and schema-change replay
- Monitor different-body reuse, stale IN_PROGRESS, expired retries, unknown states, and replay rates

## Closing

> The easy version of idempotency remembers that a key was seen. The useful version remembers what the key meant.

The second request may be a retry. It may be a different operation wearing the same key. It may be racing the first request. It may arrive after the provider succeeded but your process failed. It may arrive after your cleanup job deleted the only memory of what happened.

The server has to prove which case it is. The key is not the guarantee. The guarantee is that the server remembers the first operation precisely enough to replay it, reject a mismatch, or recover instead of guessing.
