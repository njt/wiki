# Cashpoints Partners API

Cashpoints (NZ) provides a loyalty-points API for POS integration, exposing 9 endpoints across card operations, transactional flows, and promotional queries. The API uses a lock-then-commit pattern for redemption, flat string booleans (`"Y"`/`"N"`), and returns HTTP 200 for all responses — success and failure alike.

---

## The Two-Phase Commit for Loyalty Points

The transactional flow implements a hand-rolled two-phase commit:

> "Locks a Cashpoints card balance **before** requesting payment from the cardholder"

1. **Simulate** (optional dry run) to preview points impact
2. **SimulateAndLock** to reserve the balance, returning a `cardBalanceLockID`
3. **Create** to commit the transaction using the lock ID
4. **UnlockBalance** if payment fails, releasing the lock

The lock has a **10-minute TTL**. If Create arrives after expiry, the POS must re-lock. This is pragmatic — long enough for a customer to tap their card, short enough that abandoned locks don't tie up balances.

> "We may change this in the future, which we will update in this documentation."

The honesty about the arbitrary 10-minute choice is refreshing. Most APIs would present it as immutable.

## Field Warnings as API Smell

The API documentation contains explicit warnings about field misuse:

> "**DO NOT** use this field for sale amount deduction."

This appears on `balance` and `monetaryBalance` across multiple endpoints. The correct field for discount calculation is `monetaryTotalDiscount`. The warnings suggest partners burned themselves on this repeatedly — the documentation is scar tissue from real integration failures.

It's a reminder that API design is downstream of how partners actually use (and misuse) your endpoints. The right fix would be to not return misleading fields, but the API is clearly evolved rather than designed.

## String Booleans and Flat Index Naming

The API uses `"Y"`/`"N"` strings for all boolean fields (`registered`, `active`, `fullRefund`, `itemUnqualified`). This is common in legacy POS integrations where form-encoded requests are the default.

The item schema supports two formats side by side:
- A `products` array (added in v1.3, June 2025)
- Flat `itemCode00`, `itemCode01`... fields (the original format)

This dual format is a textbook migration pattern — support the new way without breaking the old. But it creates permanent complexity. Every new integration has to pick one, and documentation has to explain both forever.

## The Refund Clawback Problem

The refund endpoint handles a real distributed-systems problem cleanly:

> "If the system cannot process point refunds directly, the partner deducts this from the cash refund to the cardholder."

When a customer has already spent points earned from a purchase, then returns the purchase, the system can't claw back points that no longer exist. Instead, `pointsToRefund` and `monetaryToRefund` go non-zero, and the partner deducts that amount from the cash refund. The API doesn't pretend the problem doesn't exist — it surfaces it explicitly and tells the partner what to do.

## PIN Handling Is Weird

The card number and PIN are concatenated into a single `cardNumber` field with an `=` separator:

```
2777640012341234=1234
```

The separator changed from `#` to `=` in v1.6 (August 2025). Concatenating authentication material with data is unusual — a separate `pin` field would be cleaner. But POS systems often have constrained input surfaces, and a single barcode scan containing both card number and PIN is likely the real constraint driving this design.

## No HTTP Error Codes

Every response returns HTTP 200. The application-level `code` integer distinguishes success from failure. This is an anti-pattern for REST APIs but pragmatic for POS terminals that may not handle non-200 responses correctly. The docs are explicit:

> "Responses have HTTP code `200 OK` regardless of `code` property."

If the service is unreachable (HTTP 500+), the body is plain text, not structured JSON/XML — a failure mode integrators need to handle separately.

## What's Missing

- **No rate limiting documentation.** POS integrations generate high volumes. Rate limits should be documented or they will be discovered at 2am.
- **No idempotency guarantees for Create.** The `partnerTransactionRef` is unique per request, but there's no stated behavior for duplicate submissions. If a POS retries after a network timeout, what happens?
- **No webhook/callback mechanism.** The flow is entirely synchronous from the POS side.
- **The Simulate endpoint has no dedicated schema page** — it points to Create's page instead. This is lazy documentation. Simulate has different constraints (reusable refs, optional lock IDs) that deserve their own reference.

---

#tool #concept

## Related

- The lock-then-commit flow is a [[Distributed Systems]] pattern — two-phase commit adapted for loyalty points
- The API's evolution (v0.1 through v1.7 over 20 months) shows organic growth without breaking changes — contrast with [[Good API Design]] principles
- The `DO NOT USE` warnings are a case study in [[Designing a Passively Safe API]] — the API can't prevent misuse, so it documents around it
- The refund endpoint's inability to claw back spent points is a worked example of [[Idempotency Is Easy Until the Second Request Is Different]] — state drifts between transaction and refund

---

*Sources: [[summary/cashpoints-partners-api-v1]]*
*Last updated: 2026-05-18*
