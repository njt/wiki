---
url: https://ferd.ca/queues-don-t-fix-overload.html
title: Queues Don't Fix Overload
author: Fred Hebert (mononcqc)
date_fetched: 2026-06-12
date_published: 2014-11-19
---

# Queues Don't Fix Overload

Hebert argues that developers frequently misuse queues as a band-aid for overload problems, when the real issue is failing to identify and respect a system's true bottleneck -- what he calls the "red arrow" -- a hard limit (database, API, disk, bandwidth, CPU, etc.) that no amount of local optimization can bypass.

## The Sink Metaphor

Hebert uses a bathroom sink analogy throughout:

1. **Normal operation**: water (data) flows in and out without issue.
2. **Temporary overload**: input exceeds output briefly; caches or buffers (queues) can handle short bursts.
3. **Prolonged overload**: the queue fills and the system crashes catastrophically.
4. **The doomed optimization cycle**: engineers optimize components, make failures rarer but more severe, then buy bigger servers, all while ignoring the fundamental bottleneck.

## Core Arguments

- Queues applied as optimizations treat symptoms, not causes. When people "blindly apply a queue as a buffer" they are "creating a bigger buffer to accumulate data that is in-flight, only to lose it sooner or later."
- Failures become less frequent but more catastrophic: "You're making failures more rare, but you're making their magnitude worse."
- The two honest choices for handling overload are **back-pressure** (blocking input) or **load-shedding** (dropping data). Both are "inescapable choices, where inaction leads to system failure."
- Real-world analogies: "Bouncers in front of a club, water spillways to go around dams, the pressure mechanism that keeps you from putting more gas in a full tank."

## The Queue-as-Optimization Trap

Typical failure cascade: an app is slow → a queue is introduced for instant speed gains → the queue overflows → more workers or persistent queues are added → the system still dies. The natural slowness was actually back-pressure keeping the system alive by limiting throughput to what the bottleneck could handle. The queue removed that safety mechanism.

Hebert notes that "the back-pressure in the system is implicit: 'tis slow," and slow distributed systems are "the canary in the overload coal mine."

## On End-to-End Principle

Persistent queues often ruin the end-to-end principle by serving as "fire-and-forget mechanisms" or assuming "tasks can't be replayed or lost." Queues introduce more points of failure -- new timeouts, new failure-detection challenges, and difficulty communicating errors back to users.

## Recommended Approach

1. Identify the true bottleneck.
2. Ask the bottleneck for permission to send more data (placing a probe to enforce back-pressure at the right point).

When load-shedding or back-pressure are properly implemented, benefits include proper QoS metrics, API designs that communicate overload honestly, fewer late-night emergencies, and more reliable endpoints for dependents. Building idempotent APIs with end-to-end principles allows callers to "safely retry requests and know if they worked."

## One Legitimate Queue Use Case

Hebert briefly notes queues as a messaging mechanism between front-end threads/processes in languages like PHP or Ruby that lack proper inter-process communication -- calling it "marginally better than using a MySQL table" but "infinitely worse than picking a tool that supports the messaging mechanisms you need."
