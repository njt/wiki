---
url: https://kerkour.com/rust-scalable-backend-services
title: "Building scalable backend services with Rust and PostgreSQL"
author: Sylvain Kerkour
date_fetched: 2026-08-14
date_published: 2026-08-12
topics:
  - software-engineering-craft
---

Sylvain Kerkour distills the patterns he uses to build medium-sized backend
services (10K+ lines, ~100 endpoints) in Rust, on top of PostgreSQL. He
concedes Rust isn't as productive as Go for backend work, but argues its rich
type system, compiler-enforced correctness, and zero-cost abstractions pay off
in fewer business-logic bugs and higher performance — and that enums alone make
it hard to go back.

The article walks the full HTTP stack top to bottom: `tokio` as the async
runtime, `rustls` (over `openssl`/`boring`) for TLS because it's Rust-native and
statically linked, `hyper` as the protocol brain that turns bytes into
`Request`/`Response`, and `axum` as the ergonomic layer on top (routing,
extractors, middleware). `reqwest` is the client-side counterpart, `tower` the
shared middleware abstraction, and `tracing` the converged observability story.

Code is organized into three layers that may only talk to their immediate
neighbors: an HTTP/scheduler/worker layer, a service layer holding all business
logic, and a repository layer that wraps database queries. Caching lives
exclusively in the service layer — the repository "must stay dumb." Background
jobs use a Postgres-backed queue, cron jobs use Postgres advisory locks for
leader election across replicas, and single-page apps are served from the same
binary via `rust-embed` to dodge the "CORS tax."

The closing advice is pragmatic: don't overthink the stack decision, just try
Rust rather than planning meetings and months of research for a 20-endpoint
service.
