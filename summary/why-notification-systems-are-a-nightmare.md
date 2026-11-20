---
url: https://notifkit.dev/articles/why-notification-systems-are-a-nightmare
title: "Why notification systems are a nightmare for engineers"
author: Notifkit (devkitshq)
date_fetched: 2026-11-20
date_published: unknown
topics:
  - software-engineering-craft
  - distributed-systems
---

A developer of the open-source notification service notifkit walks through how a one-line "send a receipt email" ticket grows, over three months, into a 25-line pseudocode `notify()` plus a workflow engine — and grounds each step in a real public incident (Salesloft, Zoho, Buttondown, HBO Max, Everbridge, among others).

The argument is a classic accretion story: a timeout demands retries; retries demand idempotency IDs, and even then the mark-sent-after-send gap means at-least-once delivery is the best you can get; provider outages demand failover, which leaks the ID across providers; user comfort demands quiet hours stored as IANA time-zone names, not offsets; bulk sends demand rate limiting with randomised backoff (a deterministic backoff caused Buttondown's duplicate newsletters), per-topic consent, and blocked lists; and the most embarrassing failures — HBO Max and Everbridge test sends to real users — are about *intent*, not recipient preference, fixed by routing all non-production traffic to a log-only provider. Finally, five messages per order fired from disconnected handlers force an order-level workflow that must survive deploys and cancellations.

The author is transparent that this is a product plug (notifkit is MIT-licensed, self-hosted on your own Postgres/Redis), and honestly disclaims exactly-once delivery: when a provider reports failure for a message it actually delivered, as with Salesloft/Gmail, no client-side design can save you. The real lesson is the closing one: no single piece is hard; they all touch each other.
