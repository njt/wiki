---
title: "Everything I Know About Good API Design"
author: "Sean Goedecke"
date: 2025-08-24
url: https://www.seangoedecke.com/good-api-design/
fetched: 2026-05-14
tags: [api-design, software-engineering]
---

# Everything I Know About Good API Design

**Author:** Sean Goedecke
**Date:** August 24, 2025

Most contemporary software engineers work extensively with APIs -- public interfaces for program communication. The author has built numerous APIs across REST, GraphQL, and command-line tools, both public and private.

## Core Philosophy: APIs Should Be Boring

Good APIs prioritize familiarity over novelty. "Any time they spend thinking about the API instead of about that goal is time wasted." Developers using APIs want tools that feel intuitive, requiring minimal documentation.

However, APIs face a unique constraint: difficulty in modification. Once published and adopted, changes break dependent systems. This creates tension -- builders must balance simplicity with long-term flexibility.

## WE DO NOT BREAK USERSPACE

Invokes Linus Torvalds' principle: avoid breaking downstream consumers. Additive changes (new fields) are acceptable, but removing or restructuring existing fields causes cascading failures across dependent code.

Responsible changes require versioning: simultaneously serving old and new API versions. Services like OpenAI (v1/chat/completions) and Stripe implement this through URLs or headers, allowing gradual user migration. However, versioning creates maintenance nightmares -- Goedecke advises using it "only as a last resort."

## Product Value Trumps Design Excellence

Paradoxically, an API's success depends primarily on underlying product desirability, not interface elegance. Facebook and Jira maintain poor APIs but command usage through market dominance. Conversely, flawless API design cannot salvage unwanted products.

Poor product architecture inevitably produces awkward APIs, since interfaces typically reflect underlying resource structures.

## Authentication Best Practices

Support long-lived API keys despite inferior security compared to OAuth. Many API consumers aren't professional engineers -- salespeople, students, hobbyists -- who struggle with complex authentication flows. Accessibility matters more than maximum security for adoption.

## Idempotency and Safety

Operations with consequences (payments, notifications) require idempotency keys -- user-defined identifiers enabling safe retry logic. Servers check whether previously processed, preventing duplicative actions.

Storage approaches vary: Redis with UUID keys works for non-critical operations; durable databases suit high-stakes scenarios. Idempotency remains optional for general use, prioritizing accessibility over perfection.

Read requests need no idempotency (duplicate reads are harmless), and DELETE operations are naturally idempotent when scoped by resource ID.

## Rate Limiting and Operational Safety

APIs enable code-speed execution, unlike UI-constrained interactions. The author recounts a Zendesk incident where third-party developers built an unintended chat system, overwhelming backend servers.

Implement rate limits with stricter constraints for expensive operations. Include metadata headers (X-Limit-Remaining, Retry-After) enabling respectful consumption.

## Pagination Strategies

Naive SELECT * approaches fail with millions of records. Page-based pagination using offsets becomes inefficient at scale -- databases must count through every offset.

Cursor-based pagination solves this: instead of offset=20, use cursor=32 (the final record's ID), converting to WHERE id > cursor. This maintains constant performance regardless of dataset size.

Always use cursor-based pagination for potentially large datasets, accepting the learning curve over future migration costs.

## Optional Fields and GraphQL

Expensive response components (subscription status requiring API calls) should be optional, controlled by parameters like include_subscription or includes arrays.

GraphQL offers maximum flexibility but Goedecke criticizes it for three reasons: high barrier to entry for non-engineers, query arbitrariness complicating caching, and fiddly backend implementation. He recommends it only when absolutely necessary.

## Internal APIs Differ

Assumptions about public APIs don't apply internally. Colleagues are typically professional engineers enabling safe breaking changes and complex authentication. However, internal APIs still require idempotency for critical operations and need incident safeguards.

## Summary of Key Principles

- APIs demand inflexibility while requiring easy adoption
- Never break public APIs; prioritize downstream stability
- Versioning enables changes but creates serious costs
- Product quality matters far more than interface elegance
- Support simple authentication for broad accessibility
- Implement idempotency for consequential operations
- Apply rate limits and operational killswitches
- Use cursor pagination for large datasets
- Make expensive fields optional; avoid GraphQL unless essential
- Internal APIs require different assumptions

The author intentionally omits REST vs SOAP discussions and format debates (JSON vs XML) as stylistic preferences rather than fundamentals. OpenAPI schemas are useful but optional -- Markdown documentation suffices.

Editorial Note: The post generated substantial discussion on Hacker News and Reddit. Commenters noted HTTP PUT's theoretical idempotency and Redis reliability concerns for idempotency stores.
