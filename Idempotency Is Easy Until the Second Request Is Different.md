# Idempotency Is Easy Until the Second Request Is Different

A deep-dive into the edge cases that make production idempotency hard. The happy-path replay cache is table stakes; the real engineering is in concurrent retries, partial failures, downstream unknowns, key reuse with different payloads, and recovery after deploys or region failovers. Dochia argues that "same key, different command" must be a hard 409 Conflict, not silent replay.

---

## Key Quotes

> "Idempotency Is Easy Until the Second Request Is Different"

The title is the thesis. Most implementations only handle the identical-retry case (same key, same payload, same request). Everything interesting happens when one of those three things changes.

> A scoped idempotency key reused with a different canonical command should be a hard error: `HTTP/1.1 409 Conflict` with `errorCode: IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_REQUEST`.

This is the article's most opinionated and useful claim. Silently returning the cached response from a *different* request is a data corruption bug dressed as a safety feature. The caller sent $100 but got the receipt for $10 and thought it succeeded.

> Hash the command, not the raw bytes. Parse into a versioned DTO, normalize values, exclude transport metadata, exclude the idempotency key itself.

Request body hashing is how you detect "same command vs. different command," but you can't hash raw bytes — whitespace, field ordering, and transport headers would break equality. Canonical serialization after normalization is the answer.

> `Redis SET NX EX` is not durable memory of the operation outcome.

Redis as an idempotency store is a footgun. If the lock expires mid-call or the process crashes after downstream success, the retry has no durable record of what happened. The idempotency store needs to survive the operation it's protecting.

> A crash after `PROVIDER_REQUEST_SENT` requires a recovery worker that queries the provider by a stable downstream key — not blindly retrying.

The recovery pattern is the hardest part. You can't just retry the provider call (double charge), and you can't just fail (the payment might have gone through). The only correct answer is a reconciliation worker that queries the provider by a deterministic downstream reference.

---

## Hard Cases Catalogued

The article systematically walks through failure modes most idempotency implementations ignore:

| Scenario | The Problem | The Fix |
|----------|------------|---------|
| Concurrent retry | Second request arrives while first is processing | Atomic ownership via `INSERT ON CONFLICT DO NOTHING`; branch on status |
| Partial local success | Crash after local commit but before event publish | Transactional outbox; recovery worker replays events |
| Downstream unknown | Provider timeout/crash after accepting | State machine + reconciliation worker with stable downstream key |
| Same key, different command | Retry with modified payload under same key | Hash the normalized command; 409 Conflict on mismatch |
| Duplicate operation without key | No natural idempotency key exists | Design one from the operation's semantic identity |
| Retry after expiry | Idempotency window lapsed | TTL must exceed maximum retry interval; alert on near-expiry |
| Retry after deploy | Code changed, stored response stale | Version the idempotency schema; invalidate on deploy |
| Region failover | Idempotency store in old region unreachable | Global table or cross-region replication for the idempotency store |

---

## Implementation Patterns

**Insert-first ownership.** The caller doesn't check-then-insert — it inserts and checks rows affected. If `rows_inserted == 1`, this request owns execution. If 0, load the existing row and branch on status.

**State machine for external side effects.** Local DB mutations are atomic. Provider calls are not. The state machine tracks progress through phases (`RECEIVED → LOCAL_PAYMENT_CREATED → PROVIDER_REQUEST_SENT → PROVIDER_CONFIRMED → COMPLETED`), and each phase boundary is a recovery point.

**Canonical request hashing.** Parse the request into a typed DTO. Normalize: strip transport headers, coerce enum casing, round timestamps to agreed precision, drop the idempotency key. Serialize with a stable serializer. Hash with SHA-256. Compare hashes to detect key reuse with different commands.

**Durable store, not Redis.** The idempotency table needs the same durability guarantees as the business data it protects. PostgreSQL with the schema above, not Redis `SET NX EX`.

---

## Key Themes

#concept #pattern #tool

---

## Critical Analysis

This is the best single-article treatment of idempotency edge cases I've seen, but it has a notable gap: **retry amplification**. When a client retries and the server's recovery worker also retries the provider, you can get N×M provider calls. The article's state machine handles crash recovery but doesn't address the coordination problem between client retries and server-side recovery.

The 409 Conflict on key reuse is the right call, but it creates a new failure mode: what does the *caller* do with a 409? The article doesn't close the loop on the client side. The caller now has a failed request, a consumed idempotency key, and no way to know whether the *first* request succeeded or failed. It needs a *new* key and a semantic check (GET the resource, compare state) before retrying. This is non-trivial.

The Redis critique is correct but incomplete. Redis *can* work as a deduplication cache in front of a durable store — the fast path checks Redis, and on miss, falls through to the database for the ground-truth check. The article is right that Redis alone is insufficient, but the pragmatic hybrid pattern deserves mention.

Compared to [[Designing a Passively Safe API]], Dochia goes deeper on command hashing and key-reuse semantics, but lighter on recovery-point checkpointing and the completer/reaper lifecycle. Albaugh's treatment of `is_transient` error classification and exponential backoff with jitter fills gaps this article leaves. Read both: Albaugh for the lifecycle, Dochia for the edge cases.

The HN discussion (16 points) was surprisingly thin — this deserves more attention. The article is doing the unglamorous work of cataloguing failure modes that most "just use an idempotency key" advice handwaves past.

---

## Related Pages

- [[Designing a Passively Safe API]] — companion treatment: recovery points, completer/reaper, transient error classification
- [[Good API Design]] — strategic overview: where idempotency matters, what to skip
- [[Software Engineering Craft]] — synthesis page placing this in the distributed systems fundamentals canon
- [[Better Error Messages]] — error design for the 409 Conflict and other idempotency error responses
- [[Frozen Test Fixtures]] — test the property not the data: idempotency testing follows the same principle
- [[Correct by Construction]] — structural correctness: idempotency as a property built into the system, not bolted on
- [[Building Production-Ready Voice Agents]] — idempotency applied to real-time voice agent infrastructure

---

*Sources: [[raw/idempotency-is-easy-until-the-second-request-is-different]]*
*Last updated: 2026-05-15*
