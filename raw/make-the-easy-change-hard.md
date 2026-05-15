---
title: "Make the easy change hard"
author: drmorr
date: 2025-08-22
url: https://blog.appliedcomputing.io/p/make-the-easy-change-hard
publication: Applied Computing Research Labs (Substack)
fetched: 2026-05-14
---

# Make the easy change hard

By drmorr, August 22, 2025. Applied Computing Research Labs.

## Summary

Inverts Kent Beck's well-known principle ("first make the hard change easy, and then make the easy change") by describing a SimKube refactoring where the author deliberately made an easy change hard -- refactoring the architecture first -- then tackled the harder implementation that followed.

## Core Problem

SimKube's tracer watches production Kubernetes cluster resources and exports changes to trace files for simulation replay. When configured to track multiple resource types (e.g., both Deployments and ReplicaSets), owned resources get duplicated: those recorded by the tracer AND those created by the controller manager during simulation. The solution requires using OwnerReference fields to distinguish root objects from owned resources, filtering out objects owned by other tracked resources before export.

## Architecture Challenges

### Arc, Mutex, and Channels

The tracer previously used `Arc<Mutex<TraceStore>>` references across multiple watchers. The Pod watcher maintained an ownership cache to avoid repeated, expensive API lookups and mutex lock contention.

Moving ownership lookup to all resources would intensify mutex lock contention. The solution: replace direct TraceStore access with message-based communication through `mpsc::Channel`.

**Original architecture:** Multiple watchers holding `Arc<Mutex<TraceStore>>` references.

**New architecture:** Watchers sending messages via `mpsc::Channel` to a message coordinator, which handles TraceStore updates. Decouples watchers from expensive operations.

### Tokio vs. Standard Library

Rust's standard library and tokio both provide concurrency primitives with different tradeoffs:
- Standard library `Mutex` cannot be held across `.await` points; `tokio::Mutex` can but at higher performance cost.
- Author notes: "tokio documentation suggests that it is 'easy' to not use them, but I spent quite a while trying to figure out a suitable way to architect it without using tokio::Mutex and eventually gave up."
- Standard library channels are not `Send`-compatible in async environments, necessitating tokio's implementations.
- `tokio::mpsc::Receiver` requires mutable self references for `recv()`, unlike the standard library version.

### Solving the Receiver Problem

Channels couldn't be stored directly in TraceStore without additional wrapper mutexes. Introduced an intermediate manager object. New constraint: tokio tasks must be `'static`, but dynamically-created channels cannot be. Solution: static function taking ownership of channel receivers.

### Export-Time vs. Ingest-Time Filtering

Initial attempts to filter owned resources during ingest failed due to message ordering -- a ReplicaSet message might arrive before its owning Deployment. Corrected approach: filter at export time. When users request a trace, the system checks ownership conditions and excludes owned resources. Introduces potential race condition between resource updates and export requests, acknowledged as rare and easily cleaned up in post-processing.

## Outcome

Two PRs: #199 (architecture refactor) and #200 (ownership filtering). SimKube 2.4.0 released. The refactoring took "multiple weeks in the refactoring mines" but resulted in improved architecture where simulation components no longer require full TraceStore construction.

## Links

- SimKube GitHub: https://github.com/acrlabs/simkube
- v2.4.0 Release: https://github.com/acrlabs/simkube/releases/tag/v2.4.0
