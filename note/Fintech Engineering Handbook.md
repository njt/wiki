# Fintech Engineering Handbook

Voytek Pitula's comprehensive pattern reference for building software that handles money — the missing textbook for fintech engineering. Organised around three unifying principles (no invented data, no lost data, no trust), it walks through representing money, recording it in ledgers, executing safe money flows, surviving the external world, and maintaining controls and access. The strongest single resource available for engineers joining fintech or building payment systems from scratch.

---

## Key Quotes

> "Money can't be created out of nowhere, so we can't tolerate duplicates or arbitrary balance updates."

The handbook's organising insight, and it's deeper than it sounds. Most bugs in financial software aren't conventional correctness bugs — they're bugs that create or destroy value. The three principles aren't just design guidelines; they're a triage framework. When you're designing a money flow, run it through each principle: could this duplicate? Could it drop a record? Am I trusting something I shouldn't? Each "no" is a defect.

> "Forbidden is not the same as unrepresentable."

One of the sharpest single sentences in the book, from the overdrafts section. A `CHECK (balance >= 0)` constraint encodes the policy but breaks when the external world forces a negative balance — and the external world will. The system that cannot represent a negative balance will crash, silently clamp to zero (inventing money), or do something similarly wrong. This is the invariant-enforcement hierarchy in action: build by construction where you can, check at runtime where you must, detect post-factum where you're forced to. Never confuse the three.

> "Assume failure every two steps."

The distilled wisdom on full resumability. A money flow is never one write — it's reserve, check, register, settle, and any of those can die mid-step. The safe assumption is that every other step will fail, and your job is to make sure a half-finished flow always lands in a recoverable state. This is the engineering version of Murphy's Law, and it has a corollary: if your system can't resume, it can't be trusted with money.

> "A webhook is an unauthenticated, unordered, possibly-lost, possibly-duplicated hint."

Pitula's webhook section is savage and correct. Don't trust the payload content; use it as a trigger to fetch authoritative state. Don't assume ordering or delivery. Acknowledge fast, process async, persist the raw bytes. This section alone is worth the price of admission — it's the checklist every webhook consumer should have taped to their monitor, and it generalises far beyond fintech.

> "The approval is part of the trail. Record who requested, who approved, and that the two were different people — otherwise the control is unprovable."

Controls without evidence aren't controls — they're conventions. An auditor can't verify intent; they can only verify records. This is the same logic that makes audit trails fundamental: the current state proves nothing; only the history of how it came to be proves everything. The four-eyes principle applies to engineering (code review, deployment approval) exactly as it applies to treasury operations — and the trail discipline is identical.

## Key Themes

### #pattern — The Three Principles as a Design Framework

The handbook's real contribution is elevating three principles (no invented data, no lost data, no trust) to a systematic design framework. Every section ends with "Principles touched:" — a traceability chain from specific pattern to governing principle. This is more than good pedagogy; it's a design review checklist. You can audit a money flow by asking three questions: does this risk inventing money? Dropping it? Trusting something it shouldn't? Each "no" maps to a specific pattern (idempotency, reconciliation, signature verification) with known implementation tradeoffs.

### #concept — Money as a Type, Not a Number

The money representation section is the best concise treatment available. Four precision models (float = never, BigDecimal = computation, integer minor-units = storage, rational = when zero loss required), the JSON serialization trap (bare numbers are IEEE-754 doubles — send strings or integers), the Money newtype pattern, controlled currency sets, and the critical insight that "pegged is not the underlying." The crypto extension (per-token precision, arbitrary-width integers for 18-decimal magnitudes) is a rare case of a fintech resource treating crypto as a first-class concern rather than an afterthought.

### #pattern — Double-Entry as Event Sourcing

Pitula frames double-entry bookkeeping as the original event sourcing pattern: balance is never stored, it's derived from the immutable event log (the ledger entries). This is a provocative connection. It means every fintech system already contains an event-sourced core — the books — and the question is how far to extend the pattern into surrounding domains. The answer: the ledger covers money; for everything else, a conventional model with a reliable change log may be enough. Event sourcing is "a very good solution when an audit trail is required, but it comes with a very high price."

### #pattern — Hold-and-Release (Funds Reservation)

The funds reservation section is unusually thorough. It covers why strong consistency is non-negotiable ("no eventual consistency here, sorry"), the available-vs-total balance distinction, why the final amount may differ from the estimate, the requirement that every reservation must eventually resolve, and the conservative failure mode (orphaned reservations lock money, they never lose or create it). Paired with the idempotency section — which covers explicit keys, error replay semantics, concurrent deduplication, time window tradeoffs, and out-of-order retry handling — this is the most complete public treatment of the two-core-mechanisms that make money flows safe.

### #concept — The Three Timestamps

Value time, booking time, settlement time. The handbook names a distinction most systems collapse into `created_at` and then can't answer audit questions about. Backdated transactions crossing reporting periods, forward-dated scheduled payments, settlement at T+2 — these aren't edge cases, they're the normal operation of money. Business reports care about value or settlement time; booking time is for traceability. The rule: record all three where they exist, and never collapse them.

### #pattern — The Invariant Enforcement Hierarchy

Three complementary approaches form a layered defense: by construction (make invalid states unrepresentable — types, constraints, smart constructors), runtime checks (assertions, property-based tests), post-factum (reconciliation, nightly checks). By construction is strongest but can't express cross-aggregate or cross-system invariants. Runtime catches violations at occurrence. Post-factum is the only one that catches bugs that already shipped — but catches them late. You use all three.

### #tool — Testing as Verification, Not Oracles

The testing appendix is a restaurant menu of techniques for money systems: property-based testing (assert invariants hold for any generated input), generative idempotency testing (auto-repeat every operation, assert zero side-effect from the second call), crash-and-resume injection (fail at every step of a flow), round-trip testing (encode/decode, serialize/deserialize), golden testing (pin outputs, diff on change), backward-compatibility testing (keep a corpus of old-format payloads), and testing in production (real money through the same books, tagged and cleaned up through normal correction machinery). The unifying idea: the invariant is the oracle, not a value you happened to expect.

## Critical Analysis

This is the book fintech engineering has needed. It's compact, opinionated, and grounded in hard-won practice rather than academic formalism. The three-principle framework gives it a spine that most domain references lack — every pattern traces back to a governing principle, which means you can reason about novel situations rather than just pattern-match against the catalog.

The handbook's greatest strength is also its limitation: it's a pattern reference, not an implementation guide. You won't find code samples, schema designs, or step-by-step tutorials. This is the right call — fintech stacks are too diverse for code-level prescriptivism to age well — but it means the book assumes significant engineering maturity. You need to know what a durable-execution engine is before the "use Temporal or hand-roll a persistent state machine" advice lands.

The crypto coverage is particularly valuable because it's rare. Most fintech resources either ignore crypto entirely or treat it as a separate domain. Pitula integrates it: crypto currencies need (network, contract address) identifiers rather than ISO 4217 codes; the precision is per-token with often-18-decimal magnitudes that overflow 64-bit integers; pegged/wrapped assets are emphatically not the underlying. This is the integration treatment the space needs.

The GDPR section is admirably practical — separate PII from financial data, crypto-shred per-user keys for embedded PII, erasure becomes key deletion — but it's notably European in frame. US engineers in money movement will find different regulatory assumptions (no federal right to erasure, different retention obligations), and the handbook doesn't flag this jurisdictional variance. A sentence acknowledging that regulatory specifics vary by jurisdiction would strengthen it.

The "circuit breakers are usually optional" take in the API consumption section is refreshingly contrarian but incomplete. Yes, circuit breakers are "mostly a courtesy toward an overloaded server," and yes, the server should handle its own load. But in practice, circuit breakers protect *your* latency and finite thread/connection pools — not just the server. The handbook mentions this in passing but underweights it. In a system where a stuck payment provider call can exhaust your connection pool and cascade into an outage, a circuit breaker isn't optional — it's infrastructure.

The book's greatest omission: it doesn't discuss observability. The audit trail is about what happened; observability is about whether the system is healthy right now. In a money system, you need both: the books to prove correctness after the fact, and dashboards/alerts to detect anomalies in real time. Reconciliation is post-factum; you also want pre-factum signals — balance drift alerts, settlement delay monitors, rate anomaly detection. The handbook's testing and reconciliation sections point at this but never name it explicitly.

These are minor quibbles with what is otherwise the single best resource for fintech engineering patterns. If you're joining a fintech company, joining a payments team, or building anything that touches money, start here. The field has needed this book for years.

---

*Sources: [[raw/fintech-engineering-handbook]]*
*Last updated: 2026-07-11*
