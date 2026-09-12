# Idempotency Is Easy Until the Second Request Is Different

The best single-article treatment of production idempotency I've read. Dochia systematically walks through every failure mode that "just use an idempotency key" advice handwaves past. The happy-path replay cache is table stakes; the real engineering is in concurrent retries, partial failures, downstream unknowns, key reuse with different payloads, and recovery after crashes.

---

## Key Quotes

> "The easy version of idempotency remembers that a key was seen. The useful version remembers what the key meant."

The article's closing line and its thesis compressed into one sentence. A replay cache remembers the key. A correct implementation remembers the scoped operation, the canonical command, the execution state, the resulting resource, the expiry window, and enough failure state to avoid turning uncertainty into duplicate side effects.

> "The second request may be a retry. It may be a different operation wearing the same key. It may be racing the first request. It may arrive after the provider succeeded but your process failed. It may arrive after your cleanup job deleted the only memory of what happened. The server has to prove which case it is."

> "The part I contest is that this is the hard part. It is not. The hard part starts with the second request, because the second request is not always a clean replay of the first one."

Most idempotency articles stop after the demo: store the key, replay the response. Dochia starts there and asks what happens when any assumption breaks.

> "A scoped key reused with a different canonical command should be a hard error, regardless of whether the first operation completed, failed, or is still running."

The article's most opinionated claim. Silently returning the cached response from a *different* request is a data corruption bug dressed as a safety feature. The caller sent $100 but got the receipt for $10 and thought it succeeded. "That is not idempotency. That is reinterpretation."

> "Hash the validated command, not the raw HTTP body."

Raw byte comparison is too strict for JSON APIs (field ordering, whitespace). But hashing has its own traps: defaults, unknown fields, locale-sensitive formatting, and fields added during deploys. "The request hash is a contract. If you change how it is computed, old retries can start looking different."

> "`Redis SET NX EX` is not durable memory of the operation outcome."

Redis as an execution guard is fine. Redis as the *only* idempotency store is a footgun. If the lock expires mid-call or the process crashes after downstream success, the retry has no durable record of what happened. "Redis can be useful. It is not a substitute for remembering the operation outcome."

> "Exactly-once delivery is not exactly-once business effect."

The queue consumer has the same bug as the HTTP handler. A message delivered twice should not send two emails, create two ledger entries, or notify a provider twice. The latter comes from durable operation IDs, unique constraints, idempotent writes, and recovery paths — not from the broker's delivery guarantees.

> "If the provider received your request and your process died before recording the result, your database cannot infer whether money moved."

The provider timeout is where your idempotency guarantee ends. The only correct answer is a recovery worker that queries the provider by a deterministic downstream reference — not blind retry, not blind failure.

---

## Status Behavior Table

Dochia's decision matrix for what the server does when it finds an existing idempotency record:

| Existing record | Same canonical command? | Behavior |
|---|---|---|
| none | yes | Insert IN_PROGRESS, execute |
| COMPLETED | yes | Replay stored response |
| any existing | **no** | **409 Conflict** — hard error |
| IN_PROGRESS, fresh | yes | Wait, 202, or 409 + Retry-After |
| IN_PROGRESS, stale | yes | Recover ownership; don't blindly re-execute |
| FAILED_REPLAYABLE | yes | Replay stored failure |
| FAILED_RETRYABLE | yes | Allow retry per policy |
| UNKNOWN_REQUIRES_RECOVERY | yes | Trigger reconciliation |
| expired/deleted | — | Follow documented expiry behavior |

The internal status set: `IN_PROGRESS`, `COMPLETED`, `FAILED_REPLAYABLE`, `FAILED_RETRYABLE`, `UNKNOWN_REQUIRES_RECOVERY`, `EXPIRED`. Don't expose every state directly, but pretending every failure is either "done" or "not done" makes recovery harder.

---

## The Schema

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

Key design decisions encoded here:
- **Scoped, not global.** `(tenant_id, operation_name, idempotency_key)` — a broken client generating `abc-123` only collides with itself. Scope might be tenant, user, account, merchant, or API client.
- **`operation_name` prevents cross-operation reuse.** A key used for `create_payment` doesn't automatically mean the same thing for `create_refund`.
- **`request_hash` is the memory of the first command.** Without it, same key + different body is ambiguous. You either replay the first response for a different command, or execute a new operation under an old key. Both are bad.
- **`locked_until` enables stale-IN_PROGRESS recovery.** Two retries can't both decide the old owner is dead — recovery ownership must be acquired atomically.

---

## Implementation Patterns

**Insert-first ownership.** Never check-then-insert. Insert with `ON CONFLICT DO NOTHING`. If `rows_inserted == 1`, this request owns execution. If 0, load the existing row and branch on status. Two concurrent identical requests: exactly one owns execution. If this test passes without a unique constraint or atomic insert, be suspicious of the test.

**Canonical command hashing.** Parse into a typed DTO → normalize (amounts, enum casing, defaults, timestamp precision) → exclude transport metadata, `Authorization`, and the idempotency key → include path params, operation name, and semantic headers (API version) → serialize canonically → hash. The hash is a contract — if you change how it's computed, old retries break.

**Outbox/inbox for events.** Same DB transaction: insert payment row + insert outbox event. Publisher delivers, consumer deduplicates by event ID or business operation key, writes side effect behind a unique constraint. Don't mark messages processed before their durable side effects exist.

**Stable downstream identities.** Each side effect gets its own durable identity:
```
client idempotency key:   abc-123
payment operation id:     payop_456
payment id:               pay_789
ledger entry key:         ledger_payment_pay_789
email dedupe key:         receipt_payment_pay_789
provider idempotency key: provider_payment_pay_789
```

**Replay contract.** Storing full response bodies gives faithful replay but retains PII, signed URLs, one-time tokens. Reconstructing from `resource_id` saves space but returns current state, which may differ from creation state. Both are valid API designs — they are not the same design. A pragmatic compromise: store `resource_type`, `resource_id`, `response_status`, `response_schema_version`, and full bodies only for endpoints where exact replay matters.

---

## Expiry and Cleanup

Idempotency records can't live forever. The replay window is a product/API decision, not just a cleanup setting. A 24-hour window means a retry at 25 hours may create a new operation.

After expiry, delete response bodies but retain metadata (key, scope, operation_name, request_hash, resource_id) for diagnostics.

**Bad cleanup:** `DELETE FROM idempotency_requests WHERE expires_at < now()` — can delete in-progress records and allow duplicate side effects.

**Better:** Small batches, partition by `expires_at`, drop old partitions, separate retention policies for bodies vs. metadata.

---

## When Not to Build This

Don't build payment-grade idempotency for admin actions where duplicates are harmless. For read-only operations, idempotency keys add noise. If duplicate analytics events cost almost nothing, a heavy idempotency table is the wrong trade. Sometimes a business key (`unique(account_id, merchant_reference)`) beats a random idempotency key. Sometimes changing the resource model (`PUT /accounts/acc_1/settings/default-currency`) makes the operation naturally idempotent.

Use the harm from duplicate side effects, the likelihood of retries, and the difficulty of detecting duplicates after the fact to decide how much machinery you need. "If duplicates move money, notify humans, call providers, consume scarce inventory, or corrupt accounting, spend the design effort."

---

## Testing and Monitoring

Tests worth more than a dozen happy-path unit tests:

1. Same key, same command, completed → replay, no second side effect
2. Same key, different command → 409 Conflict
3. Two concurrent identical requests → one wins, side effect fires once
4. Timeout after downstream success → retry recovers, doesn't call provider with new identity
5. Duplicate queue message → one ledger entry, one email, one provider notification
6. Expired/stale state, schema-change replay, cross-region retry

Metrics that find bugs (not just capacity plan):
- `idempotency.conflict.different_request.count` — client bugs
- `idempotency.in_progress.age.max` — stuck workers
- `idempotency.expired_retry.count` — window too short
- `idempotency.unknown_state.count` — recovery gaps

---

## Critical Analysis

This is the best single-article treatment of idempotency edge cases I've seen, but it has gaps.

**Retry amplification.** When a client retries and the server's recovery worker also retries the provider, you can get N×M provider calls. The state machine handles crash recovery but doesn't address coordination between client retries and server-side recovery.

**Client-side 409.** The 409 Conflict on key reuse is the right call, but what does the *caller* do with it? The caller now has a failed request, a consumed idempotency key, and no way to know whether the *first* request succeeded. It needs a new key and a semantic check (GET the resource) before retrying. Dochia doesn't close this loop.

**Redis hybrid.** The Redis critique is correct but incomplete. Redis *can* work as a deduplication cache in front of a durable store — fast-path check Redis, fall through to the database for ground truth. Redis alone is insufficient, but the pragmatic hybrid pattern deserves mention.

Compared to [[Designing a Passively Safe API]], Dochia goes deeper on command hashing and key-reuse semantics, but lighter on recovery-point checkpointing and the completer/reaper lifecycle. Albaugh's treatment of `is_transient` error classification and exponential backoff with jitter fills gaps Dochia leaves. Read both: Albaugh for the lifecycle, Dochia for the edge cases.

---

## Related Pages

- [[Designing a Passively Safe API]] — companion: recovery points, completer/reaper, transient error classification
- [[Good API Design]] — strategic overview: where idempotency matters, what to skip
- [[Software Engineering Craft]] — synthesis page for distributed systems fundamentals
- [[Better Error Messages]] — error design for 409 Conflict and other idempotency responses
- [[Correct by Construction]] — idempotency as a property built in, not bolted on

---

*Sources: [[summary/idempotency-is-easy-until-the-second-request-is-different]]*
*Last updated: 2026-05-18*
