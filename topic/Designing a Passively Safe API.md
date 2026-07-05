# Designing a Passively Safe API

Dane Albaugh's deep-dive into making APIs that can't hurt you when things go wrong. The core idea: after any failure, the system either completes the workflow exactly once, or lands in a terminal, explicitly visible state. No duplicate charges, no orphaned side effects, no mystery. This isn't aspirational -- it's a concrete engineering pattern built from idempotency keys, transactional outbox/inbox, and recovery-point checkpointing.

---

## Key Quotes

> "Failures (crashes, timeouts, retries, partial outages) can't produce duplicate work, surprise side effects, or unrecoverable state."

> "Submitting the same request multiple times should have the same effect as submitting it once."

> "Exponential backoff with jitter to prevent overwhelming the server."

## Key Themes

#api-design #error-handling #sre #simplicity

The article breaks endpoints into "atomic phases" -- groups of local mutations in a transaction, separated by foreign state mutations. Each phase has a recovery point, so if the server crashes between phase 3 and phase 4, a retry resumes from phase 3 rather than starting over and duplicating work.

Three patterns do the heavy lifting: the **outbox** (messages inserted within the transaction, drained by a background worker -- guarantees at-least-once publish), the **inbox** (deduplication on the receiving end via unique message IDs), and **idempotency keys** (client-provided keys that let the server return cached results on retry).

The error handling is particularly sharp: include an explicit `is_transient` boolean in error responses instead of making clients guess from HTTP status codes. Transient errors are not cached; non-transient errors are. A "completer" process retries stuck requests, and a "reaper" cleans up terminal keys after 72 hours.

This connects directly to the operational reliability themes in [[The Future of Software Engineering is SRE]] -- passive safety is how you make the "other 190%" survivable. It also echoes the distributed systems principles in [[Building Production-Ready Voice Agents]], where every API call needs timeouts, circuit breakers, and graceful degradation.

## Critical Analysis

This is the best single-article treatment of API idempotency I've seen. It goes beyond "just add an idempotency key" to cover the full lifecycle: recovery points, completer/reaper processes, request body hashing to catch mutation-by-retry, and explicit transient vs. non-transient error classification. The five-phase shipping example makes the abstract concrete.

What's missing: no discussion of how this pattern interacts with event sourcing or CQRS, which are natural companions. Also no mention of the operational cost -- maintaining completer and reaper processes is real infrastructure. But for a focused article, the scope is right.

---
*Sources: [[summary/designing-a-passively-safe-api]]*
*Last updated: 2026-05-14*