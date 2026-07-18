---
url: https://www.patreon.com/engineering/posts/how-we-scaled-162544709
title: How We Scaled Notifications with Fanout
author: Lingene Yang (backend engineer at Patreon)
date_fetched: 2026-07-18
date_published: 2026-07-15
site: Engineering at Patreon
---

# How We Scaled Notifications with Fanout

**Author:** Lingene Yang (backend engineer at Patreon)

## Summary

The article describes Patreon's rebuild of their notification platform around a fanout architecture to address severe performance bottlenecks that emerged by early 2025 as creator audiences grew dramatically, particularly after free memberships launched.

## Legacy System Limitations

The original flow used a single asynchronous task that fetched all recipients, filtered by notification settings, generated payloads, and handed them to delivery systems. For the largest creators, this single task handled millions of notifications and "consistently timed out" by early 2025.

Two major limitations are cited: lack of horizontal scalability (all work inside one task) and no channel isolation (in-app feed, push, and email were tightly coupled so "a failure in one channel could block the others").

## Fanout Platform Design

The platform was designed around six requirements: developer-friendly, horizontally scalable, isolated channels, priority-based processing, observable, and extendible.

### Notification Factory Abstraction

A factory pattern separates platform orchestration from notification-specific business logic. The **Notification** model (task arguments) links to a subclass of **BaseNotificationFactory** (task handler) containing configuration for filtering and generating notifications. A registry maps each notification model to its factory class.

### Single API

The team replaced separate channel-specific APIs with a unified `send_fanout_notifications` API. Callers pass a notification payload, recipient list, and a timestamp for latency measurement. The platform handles batch sizing, priority determination, and channel-specific fanout.

### Two-Stage Fanout Architecture

1. **First stage:** Splits the full recipient list into batches; each `FanoutNotifications` task filters recipients, runs eligibility logic, and generates channel-specific payloads.
2. **Second stage:** Fans out into dedicated delivery tasks for in-app feed, push, and email.

## Observability Improvements

The platform introduced timing/logging data models that propagate through the system, recording timestamps at each stage and emitting metrics for both aggregate and per-recipient outcomes. This addressed the legacy problem where investigations "could take engineers several hours."

## Migration: 200+ Notifications

The team built AI skills grounded in documentation and exemplary PRs, triggered via `/migrate-notif-fanout <notif_name>`, to handle the repetitive structure. AI was "not a replacement for engineering judgment" but significantly accelerated the work.

The migration initially progressed slowly (20% in 6 months via bottom-up prioritization). The turning point came with engineering leadership alignment to finish by Q1 2026, after which the remaining migrations completed in 6 weeks. The effort involved 10 teams and over 30 engineers, with a migration week event featuring office hours, leaderboards, prizes, and a happy hour.

## Performance Results

- For large creators: push and in-app feed notifications are **80% faster**; email is **55% faster**.
- When a creator's audience grows 4x, end-to-end latency now increases "by 33% for push and in-app feed and 60% for email," versus a "186% increase" on the legacy platform.

## Lessons Learned

1. **Large migrations need explicit prioritization** — leadership alignment was critical; without clear deadlines, cross-team migrations compete with every team's roadmap.
2. **Platform work does not end at the API** — improving observability, alerting, and debugging tools enabled product engineers to investigate independently and reduced the on-call burden.
3. **Design for the next bottleneck, not just the current one** — recipient list generation is the next target, and the platform was designed to accommodate that future capability.

## What's Next

The team plans to bring recipient list generation into the notification factory for end-to-end ownership, and is improving notification settings for both developers and fans.
