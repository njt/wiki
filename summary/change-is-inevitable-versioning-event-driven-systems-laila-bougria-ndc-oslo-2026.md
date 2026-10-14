---
url: https://www.youtube.com/watch?v=kCtal3G1vkE
title: "Change is inevitable: versioning event-driven systems"
author: Laila Bougria
date_fetched: 2026-10-14
date_published: 2026
topics:
  - distributed-systems
  - software-engineering-craft
---

Laila Bougria's NDC Oslo 2026 talk (62 min) argues that the blind spot in event-driven architecture is not boundaries, sync-vs-async, or orchestration — it is **change**. Everything in the talk serves one thesis: versioning is designed in up front or it fails in flight.

The opening case study is a credit card transaction service emitting one fat `CreditCardTransactionMade` event with an `operationType` discriminator. Every natural evolution — reserving hotel funds, adding withdrawal costs, currency questions — forces consumers to learn the producer's internal rules ("functional coupling") and eventually to break deserialization entirely: remove a field consumers assumed required and you manufacture poison messages that retry forever. Her fix: granular, business-meaningful event types (`PaymentMade`, `RefundIssued`, `FundsBlocked`, `CostIncurred`), one meaning each, stable over time, each carrying only relevant attributes.

She then splits **public vs private events**: private events are granular, change-prone, rich in domain context, and safe because only your own team consumes them; public events must be self-contained, stable, and minimal — because once emitted, like a photo on the internet, you lose control of who screenshots them. Summary events (`CardPeriodClosed`) let a service encapsulate aggregation logic behind a stable contract. Design buys a smaller **change radius**.

On the wire, she distinguishes format coupling (serialization) from the implicit assumptions beyond it — decimal precision, required-vs-optional, units (the Mars Climate Orbiter loss as the cautionary tale) — and prescribes explicit schemas with a chosen compatibility mode: forward (producers upgrade first; short-lived broker messages), backward (consumers upgrade first; long retention like Kafka), full, and transitive. She carefully separates schema versioning from semantic versioning: deleting a field can be a non-breaking schema change even though it is a semver major.

Metadata changes are always breaking — the CloudEvents spec standardizes the envelope (id, type, source, plus data schema/content-type/time), and any change to it warrants a new event type. For unavoidable breaking changes she gives an upgrade playbook: deprecate with a removal date, dual-publish through the window, then stop — and the migration mechanics (subscribe to the new topic *before* killing the old subscription, use false filters to drain in-flight messages, deduplicate by logical message id). Non-atomic unsubscribe-then-subscribe loses messages at high throughput.

Closing: compatibility guarantees are "just promises," and quoting Clemens Vasters — distributed systems become "absolute chaos if breaking our promises is easy." Schema registries formalize the promise but are mostly broker-locked (Confluent⇒Kafka); she champions the CNCF **xRegistry** spec (still RC4) as protocol-agnostic registry of endpoints, message definitions and schemas, whose registry-change events finally let producers notify consumers without knowing who they are — restoring the decoupling pub/sub promised in the first place.

Takeaways: design events for change up front, pick a compatibility mode as a public policy, structure the envelope with CloudEvents, enforce with a registry, and have an upgrade strategy before the first breaking change.
