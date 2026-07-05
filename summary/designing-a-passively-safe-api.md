---
title: "Designing a Passively Safe API"
url: https://www.danealbaugh.com/articles/passively-safe-apis
date_fetched: 2026-05-14
section: "Producing and Operating Software"
---

A passively safe system "is designed to fail gracefully." In APIs, this means "failures (crashes, timeouts, retries, partial outages) can't produce duplicate work, surprise side effects, or unrecoverable state."

Uses a POST /shipments endpoint to illustrate common failure modes: external API calls outside transactions, non-idempotent requests, synchronous dependencies, message delivery gaps.

Core solutions: Message Outbox Pattern (insert messages into DB within transaction, background worker drains and publishes), Message Inbox Pattern (store incoming messages with unique constraint on message ID), Idempotency Keys (clients provide unique key per request, server stores recovery points).

Break endpoints into "atomic phases" -- groups of local mutations in transactions separated by foreign state mutations. Recovery points checkpoint progress between phases, enabling resumption on retry. Example: five phases from creating idempotency key through to finalizing and enqueueing notifications.

HTTP semantics: GET, PUT, DELETE are idempotent by definition. POST creates resources (not idempotent by default). PATCH performs conditional updates (not idempotent by definition).

Include explicit is_transient boolean in error responses. Use exponential backoff with jitter. Implement "completer" process for stuck requests and "reaper" for terminal keys after ~72 hours. Hash request bodies to reject retries with different payloads but same idempotency key.

The design handles: address validation failures, lost responses, concurrent duplicate requests, external API downtime, notification service outages, server crashes, breaking API changes as isolated phase failures.