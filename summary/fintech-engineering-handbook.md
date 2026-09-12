---
url: https://w.pitula.me/fintech-engineering-handbook/
title: "Fintech Engineering Handbook"
author: Voytek Pitula
date_fetched: 2026-07-11
date_published: 2026-06-29
topics:
  - software-engineering-craft
---

A patterns handbook for building software that handles money, organised around three principles: **no invented data** (idempotency, deduplication, reconciliation), **no lost data** (full precision, at-least-once delivery, event sourcing, immutability), and **no trust** (verify webhooks, cross-check data sources, fail loudly on broken assumptions).

The handbook walks through the stack from bottom to top. It starts with representing money — precision strategies (floating-point is almost never right; minor-unit integers for storage, arbitrary precision for computation), explicit rounding, and currency handling that pairs amount with currency and forbids cross-currency arithmetic. FX rates are directional, non-invertible, and source-dependent.

Recording money centres on the double-entry ledger: every movement has source and destination, balances are derived not stored, and entries are immutable — corrections are compensating postings, not edits. The audit trail captures what happened, when, who triggered it, and why. Event sourcing turns the trail into the primary artifact. GDPR is handled by separating PII from financial data and using crypto-shredding for embedded personal fields.

Executing money flows covers invariants (enforced by construction, at runtime, and post-factum), funds reservation (hold-and-release to prevent double-spend), intentional vs. unintentional overdrafts (never encode "never negative" at the type level — the external world can force one), idempotency keys scoped to operation and client, and full resumability via durable state machines.

The external world section treats providers as inherently unreliable: validate schemas at the boundary, persist every request and response, treat webhooks as hints not authoritative state, and reconcile independently. The outbox pattern and CDC are the practical ways to publish events reliably. Reconciliation is the safety net that catches the dropped facts.

Controls turn the "no trust" principle inward: segregation of duties, maker-checker for sensitive operations, least-privilege access with audited grants and periodic recertification, and a verifiable SDLC trail from commit to deploy.

Testing emphasises property-based tests over enumerated cases — invariants as oracles, generative idempotency testing, crash-and-resume injection, round-trip tests for precision, golden tests for gnarly computations, and backward-compatibility tests for old-format payloads. Testing in production moves real money and must go through the same ledger, tagged and reversible.

The appendices provide a domain glossary (ledger, general vs. sub-ledger, chart of accounts, suspense accounts, write-offs, commingling) and sourcing notes. The handbook is written for people joining fintech, already in fintech, or outside it — a shared vocabulary and a reference to reach for when facing a particular problem.
