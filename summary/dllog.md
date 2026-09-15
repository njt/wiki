---
url: https://github.com/arhuman/dllog
title: "dllog"
author: arhuman
date_fetched: 2026-09-15
date_published: 2026-09-12
topics:
  - software-engineering-craft
  - developer-tools
---

dllog is a Go library (v0.1.0, MIT, ~2,000 lines of non-test Go) that dissolves the oldest trade-off in production logging per operation rather than per service: `Debug` floods production, `Info` hides the context that explains an error. It sits between your code and your existing logger — it is a `log/slog.Handler` in the root package and a `zapcore.Core` in `zapadapter/`, not a logging library of its own. Records between the buffer floor (Debug) and the effective level (Info) are buffered inside a per-operation scope; a successful operation discards them and stays as quiet as Info, a failed one replays them — marked `replay=true`, with their original timestamps, through the downstream they were logged with — ahead of the error that released them.

The machinery is a bounded drop-oldest ring buffer per scope (256 records by default), with a one-shot "trip" that flushes it. HTTP services get this for free from `dllog.Middleware()`, which opens a scope per request and trips on 5xx or panic, emitting a single anchor error record first; non-HTTP code opens a scope with `dllog.Scope(ctx)` and calls `dllog.Trip(ctx)` at the failure site, covering the common Go case where the error is returned rather than logged.

The cost model is the point: outside a scope a below-level call costs 9.1 ns/op and zero allocations against 4.2 ns for a plain Info-gated slog logger — the record is refused before it is built, and the difference is one context lookup. Inside a scope, buffering costs about 235 ns and one allocation, an insurance premium paid only on operations that might fail. The buffer is count-bounded, never grows with uptime, and ring arrays are recycled through a pool.

The implementation is unusually disciplined for a v0.1: a generic engine in `internal/core` that imports no logging library (enforced by a CI script), two independent adapters sharing one ring per context so a trip through zap replays what slog buffered, three ADRs dated the release day explaining the replay key and context-retention decisions, benchmark numbers kept as a single source of truth behind a `make` target, and a soak test driving 60,000 scopes and 2.4 million records while asserting the heap stays flat.
