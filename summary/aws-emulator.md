---
url: https://github.com/aws/aws-emulator
title: "AWS Emulator (aws/aws-emulator)"
author: Amazon Web Services
date_fetched: 2026-11-13
date_published: 2026-11
topics:
  - developer-tools
  - specifications-as-the-product
---

AWS Emulator is Amazon's open-source local emulator of the AWS APIs: one process serving a self-signed HTTPS endpoint on 127.0.0.1:9297 that answers requests the way AWS would, so software can be tested without an AWS account. It is built as a Rust workspace with **one crate per AWS service — 437 service crates, ~1.16M lines of Rust** — mounted by a single router crate, plus an embedded-mode crate that runs the whole thing in-process for Rust tests. Released under CC0 (vendor crates Apache-2.0); no PRs accepted.

The load-bearing idea is that **the Smithy API model is the product**. Each service crate carries a copy of the service's official Smithy model (DynamoDB's alone is ~950KB of JSON), and its `build.rs` generates the wire protocol, routing table, and operation scaffolding from that model at compile time. Humans/agents hand-write only behavioral seams: `service.rs` (routing claims, error wording), `state.rs` (the in-memory store), and one file per operation under `ops/`. Every generated operation answers 501 until a handler is written, so fidelity gaps are always visible rather than silently wrong.

Two front doors share one dispatch path: the standalone binary (`aws-emulator-router`) and `AwsEmulatorEmbedded`, which hands the SDK an in-process `HttpClient` and minted credentials, with a fresh engine per instance so tests cannot leak state into each other. The vendored framework enforces house rules at compile time: no `unsafe`, no `todo!()` (a panic renders as a 500 the real AWS never sends), and handlers must run user code under a wall-clock timeout and step budget with no outbound network, since in-process execution has no isolation boundary.

Fidelity is managed in public: each release publishes per-service fidelity scores, and the repo tracks ~6,900 open gaps in `issues/things-that-do-not-work/` with ~13,200 fixed issues moved to `things-that-now-work/`. The README's quick-start starts with installing an agent skill (`aws agent-toolkit add-skill`), positioning the emulator as the environment coding agents test AWS-shaped software against.
