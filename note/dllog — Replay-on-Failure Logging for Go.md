# dllog — Replay-on-Failure Logging for Go

dllog ([arhuman/dllog](https://github.com/arhuman/dllog), v0.1.0, September 2026) is a ~2,000-line Go library that breaks the classic production logging trade-off per operation instead of per service. The trade-off: run your logger at `Info` and a failed operation arrives with no context — the card token, the retry, the decline code were all below the level — or run it at `Debug` and pay for verbose logging on every successful request too, which is why nobody leaves it on. dllog buffers below-level records in a bounded per-operation ring and replays them, marked `replay=true` with original timestamps, only when the operation fails. It is not a logging library and replaces nothing: it is a `log/slog.Handler` and, separately, a `zapcore.Core` that plug into whatever you already run — even both mixed in one service, sharing one buffer.

---

## Architecture

Three layers, with a hard dependency wall in the middle:

**The engine — `internal/core`.** A generic package that imports no logging library (CI-enforced by `scripts/check-core-imports.sh`), because that constraint is what lets one engine back several adapters. Its three abstractions:

- `Scope[T]` (`internal/core/scope.go`) — the per-operation state machine: a fixed preallocated ring (256 slots default, evict-oldest-when-full), a one-shot `Trip()` that hands buffered entries back in insertion order, and an explicit `Close()`. `Append` returns a tri-state `Action` — `Buffered`, `PassThrough`, or `Suppressed` — so the caller owns all emission and the engine never touches a logging API.
- `Carrier` (`internal/core/carrier.go`) — the context-attached shared state: lazy ring binding, a `preTripped` flag for trips that arrive before anything buffered, and a permanent `closed` flag. Every per-record check (`is there a scope / is it closed / has it tripped`) is an atomic load; the mutex guards only transitions.
- `ScopePool[T]` (`internal/core/pool.go`) — ring recycling, so a service opening one scope per request stops allocating rings at steady state.

**Two independent adapters over that engine.** The root package is a `log/slog.Handler` (`handler.go`); `zapadapter/` is a `zapcore.Core` (`zapadapter/core.go`). Neither is built on the other — both store their own slot type as a `core.Entry` in the *same* ring on a given context, so an `Error` logged through zap replays what slog buffered and vice versa, in append order. The asymmetry worth noting: slog carries context per call, so the handler finds the scope in `Handle(ctx, …)`; zapcore has no context in `Check`/`Write`, so `zapadapter` requires an up-front `base.For(ctx)` to produce a bound Core, and an unbound Core is just a level filter.

**The HTTP front door — `middleware.go`.** `dllog.Middleware()` opens a scope per request, wraps the writer in a `statusWriter`, and trips on 5xx or on panic (recovering only long enough to trip, then re-panicking the original value — "a logging tool, not a recovery layer"). On trip it emits one anchor `Error` — `"dllog: request failed"` with method, route pattern, and status — so a replay is never a headless pile of Debug lines.

## Key techniques

- **The three-level model.** `bufferFloor ≤ level ≤ tripLevel` (Debug ≤ Info ≤ Error by default), validated with a panic at construction. `Enabled()` answers from the handler's own config and *never consults the downstream* — delegating made every out-of-scope buffer-band call build a full record that `Handle` then threw away. Answering locally keeps an out-of-scope Debug at 9.1 ns/op, zero allocations, versus 4.2 ns for a plain disabled slog call; the delta is one context lookup. The honest baseline it beats: 154.9 ns to actually render Debug everywhere.
- **One allocation per buffered record.** Each ring slot is a small struct implementing `core.Entry` with pointer methods — the root slot is `{ctx, record (cloned), downstream handler, replayKey}`. The doc comments are explicit that this replaced "two escaping closures per entry" on the hottest path; the interface word carries the pointer without boxing.
- **Replay fidelity as a first-class decision (ADR 0001, ADR 0002).** The replay key is captured *per entry at append time*, not chosen at flush — because the key belongs to the handler that logged the record. The old trip-time model had three trip paths marking the same batch with different keys, and the middleware (which holds no handler) silently broke `WithReplayKey`. Each slot also keeps the original `context.Context`, so trace/span IDs and tenant attrs survive onto replayed records — cancelled or not, per slog's read-values-never-cancellation contract. The retention objection doesn't hold because the ring clears at trip/close, which coincide with the end of the operation the context belongs to anyway.
- **The post-trip budget bounds everything, not just replays.** After a trip, pass-through records *at* the effective level draw on the same `WithPostTripLimit` budget as replayed entries — so the budget caps a scope's entire post-failure output, which is what an error storm following the failure actually needs. The triggering record itself is never suppressed: it's the anchor the batch hangs from.
- **Drop-oldest is right for diagnosis.** When the 256-slot ring fills, it evicts the oldest, keeping the records nearest the failure — the diagnostic ones — and a synthetic record (`dllog.dropped=N`) is emitted before the batch so truncation is never silent.
- **A closed carrier stays closed.** The context outlives the scope, so records keep arriving after `done()` runs; they are suppressed, exactly as a scope-less below-level record would be, and no ring is ever re-bound. Combined with join-not-nest `Scope` semantics (an inner `Scope` returns the same scope and a no-op `done`), the ownership story stays simple: exactly one creator releases one buffer.
- **A pooling trick for Go's missing generic vars.** Go has no generic package-level variables, so the `sync.Pool` hangs off the long-lived Handler/Core, and rings are pooled as `*[]T` *boxes* — a pointer fits in an interface word, so `Put` doesn't allocate to box a slice header. The whole box (array + header) is recycled; arrays are zeroed on put.
- **Mis-wiring panics at startup.** `New(downstream)` probes `downstream.Enabled(bufferFloor)` and panics if the downstream would filter — because a filtered downstream silently discards exactly the replays the library exists to deliver. Constructors also panic on out-of-order levels. Fail at setup, not at 3 a.m.

## Design decisions

The central inversion is economic. Conventional debug logging is a subscription: pay ~155 ns per record on every operation, forever, to have context when the ~0.1% failure arrives. dllog is an insurance premium: pay ~235 ns and one allocation per record, only inside operations that might fail, and collect the full Debug story exactly when it's needed. The middleware numbers bracket it: a clean 200 costs 2,231 ns/op (21 allocs), a failing 500 costs 5,137 ns/op — the replay is microseconds, once, per failure.

The trade-offs are declared, not hidden, in `docs/caveats.md`:

- **Write-now/format-later.** Log arguments are kept as references and formatted only if replayed. Mutate a pointer, slice, or map after logging it and the replay shows the *current* value, not the one at log time. The fix is guidance ("log ids and strings"), not defensive deep copies — which would tax every buffered record for a corruption case the caller can avoid.
- **Count-bounded, not byte-bounded.** `WithCapacity` caps records; their retained size is uncapped. "A bounded count of large objects is still large" is the doc's own phrasing. What keeps this honest is the soak suite: `TestSoakScopeChurnKeepsHeapFlat` drives 60,000 scopes and 2.4 million records and asserts the heap stays flat — and the file keeps a *mutation test* proving that bound can actually catch a leak.
- **The `WithGroup` marker moves.** The replay marker is added at flush time, so under a grouped handler it nests inside the group instead of sitting at the record root. Documented rather than engineered around: matching slog's grouping rules to hoist it costs more than tidier output is worth.

The documentation discipline is itself a design artifact: three ADRs dated the release day, a performance doc that is "the single source of truth" with figures regenerated only via `make bench`, and constructor panics replacing silent misconfiguration. For a v0.1.x, the ratio of explained decisions to lines of code is remarkable.

## Comparison notes

- [[Reduce Logging Costs]] — Shpilt's five-strategy taxonomy assumes the trade-off dllog dissolves: you either emit debug everywhere or lose it, so his ladder runs from sampling down to "killing INFO logs," which he rightly calls surrender. dllog is the in-code strategy taken past reduction into *deferral*: emit nothing extra on success, and pay per buffered record only inside operations that might fail. It converts Shpilt's destructive last resort into a non-destructive default.
- [[Wide Events vs. Three Pillars — AI Observability Costs]] — Honeycomb's answer to failure-path poverty is to pre-pay one rich structured event per operation. dllog keeps the line-based model but materializes the equivalent *lazily*: the replayed batch is a post-hoc wide event assembled, with original timestamps, only on the failure path. Deferred instead of pre-paid — the same destination, opposite cash flow.
- [[The Log — Unifying Abstraction for Real-Time Data]] — the same noun, opposite economics. Kreps' log is append-only and kept forever, the substrate of replication and integration; dllog's ring is append-only for the duration of one operation and usually discarded. One abstraction's whole value is permanence; the other's is disposability.
- [[DDB — Source-Level Interactive Debugging for Distributed Applications]] — DDB extends interactive stepping into distributed systems at 1–5% throughput overhead; dllog is its passive, log-shaped cousin — no breakpoints, no live inspection, just enough per-operation history (≤256 records at ~235 ns each) that the failure explains itself after the fact. Both refuse to change how you write code in exchange for failure context.

There is also a database parallel the project never claims: buffer-then-replay-on-failure is structurally the write-ahead log idea — record the detail first, replay it in order if recovery is needed — as in [[ARIES — Write-Ahead Logging Recovery]], except the "crash" is an ordinary error return and the recovery is one log batch.

## Verdict

The clever insight is that the interesting quantity in logging is not volume but *timing*: the same Debug records are worthless on a success and precious on a failure, so the only rational price to pay for them is one conditioned on failure. dllog prices it correctly. The obvious weaknesses — retention is bounded by count not bytes, replay fidelity depends on callers not mutating what they logged, and everything runs synchronously on the calling goroutine — are all real but all chosen, documented, and cheaper than the alternatives. What a senior engineer will envy is not the ring buffer (which is conventional) but the surrounding honesty: panics on mis-wiring, benchmarks as a regenerable source of truth, and caveats written as warnings rather than footnotes.

#tool #project #observability #debugging #golang

---
*Sources: [[raw/dllog]], [[summary/dllog]]*
*Last updated: 2026-09-15*
