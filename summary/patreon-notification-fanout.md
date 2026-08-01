---
url: https://www.patreon.com/engineering/posts/how-we-scaled-162544709
title: "How We Scaled Notifications with Fanout"
author: Lingene Yang (backend engineer at Patreon)
date_fetched: 2026-07-18
date_published: 2026-07-15
---

Patreon rebuilt their notification platform around a fanout architecture after the legacy system — a single async task handling millions of notifications for large creators — began timing out consistently by early 2025. The old design lacked horizontal scalability and tightly coupled in-app feed, push, and email delivery so that a failure in one channel could block the others.

The new platform splits work into two stages: a first stage batches recipients and runs eligibility logic, and a second stage fans out into dedicated delivery tasks per channel. A factory abstraction decouples platform orchestration from per-notification business logic, and a unified `send_fanout_notifications` API replaced separate channel-specific APIs.

The team migrated over 200 notification types, initially slow (20% in 6 months) until engineering leadership aligned on a Q1 2026 deadline, after which the remaining migrations finished in 6 weeks with involvement from 10 teams and 30+ engineers. AI-assisted tooling accelerated the repetitive work but did not replace engineering judgment.

Results: push and in-app feed notifications are 80% faster for large creators; email is 55% faster. Under 4× audience growth, end-to-end latency rose only 33–60% versus a 186% increase on the legacy platform. The team also invested in observability, cutting investigation time from hours to self-serve for product engineers.

Key lessons: large migrations need explicit leadership prioritization; platform work extends beyond the API to observability and debugging tooling; and the architecture should accommodate the next bottleneck — here, recipient list generation — not just the current one.
