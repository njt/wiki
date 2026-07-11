---
url: https://github.com/satmihir/grudge
title: grudge — A constant-memory sketch that holds grudges and forgives
author: Mihir Sathe (satmihir)
date_fetched: 2026-07-11
date_published: 2025
---

# grudge

A constant-memory decaying-score sketch for Go: maps an unbounded key space to a scalar score that updates with feedback and decays toward zero on its own. It is to behavioral scores what a count-min sketch is to counts. Extracted from the FAIR project (Stochastic Fair BLUE-based fairness throttling).

## Architecture

The project is a single Go package (~1,860 lines of source) with one internal subpackage and one exported test-helper package:

```
grudge/                 — package grudge: Sketch, Rotator, Decay, Aggregator, Config, tuning
  internal/hash/        — Hasher interface, Murmur3 + SipHash implementations
  grudgetest/           — FakeClock, FakeTicker for deterministic testing
```

### Core types

- **`Sketch`**: L levels × M cells, one hash seed per sketch. Each key maps to one cell per level via Kirsch-Mitzenmacher hash derivation. All operations touch exactly L cells — O(L) independent of key count.
- **`cell`**: `{score float64, lastUpdated int64 (UnixMilli), sync.Mutex}` — the only mutable state.
- **`Decay`**: interface with `Apply(score, dtMillis) float64`. Built-ins: `Exponential(λ)`, `Linear(rate)`, `None`.
- **`Aggregator`**: `func([]float64) float64`. Built-ins: `Min`, `Max`, `Mean`.
- **`Rotator`**: manages N generations under different hash seeds, reading from the oldest and writing to all, with periodic rotation to bound false-positive lifetime.

### Key files

| File | Lines | Role |
|---|---|---|
| `sketch.go` | 263 | Sketch struct, New, Update, Query, UpdateAndQuery, TryUpdate, QueryDetailed |
| `decay.go` | 114 | Decay interface, Exponential, Linear, None, validation |
| `aggregator.go` | 40 | Min, Max, Mean aggregators |
| `config.go` | 59 | Config struct with validation |
| `rotator.go` | 168 | Rotator: multi-generation management with background goroutine |
| `tune.go` | 56 | SuggestLevels: sizing from collision-probability formula |
| `hash.go` | 42 | Hasher/HasherFactory interfaces, Murmur3/SipHash factories |
| `clock.go` | 28 | Clock/Ticker interfaces with real implementations |
| `internal/hash/hash.go` | 27 | Murmur3 wrapper, Splitmix64 |
| `internal/hash/siphash.go` | 89 | Full SipHash-2-4 implementation |
| `grudgetest/clock.go` | 57 | FakeClock, FakeTicker for deterministic tests |

### Dependencies

- `github.com/spaolacci/murmur3` — the default hasher
- `pgregory.net/rapid` — property-based testing framework (test-only)
- `go.uber.org/goleak` — goroutine leak detection (test-only)

## Three defining properties

1. **Collision shielding**: With Min aggregation, a key's estimate is exact whenever at least one of its L cells is free of colliding traffic. Error is one-sided (`Query(k) ≥ true score`). False-positive probability: `(1 - (1 - 1/M)^H)^L`.

2. **Lazy decay**: Scores fade toward zero, computed on access only. An idle sketch does zero background work. Decay must compose across time subdivision: `Apply(Apply(s, t1), t2) == Apply(s, t1+t2)`.

3. **Rotation**: A Rotator keeps N generations under different seeds, reads from the oldest, writes to all (so successors are warm), and periodically retires the oldest. Worst-case false positive lasts at most `Generations × Period`.

## Operations

| Op | Description | Locking |
|---|---|---|
| `Update(k, δ)` | Add δ to all L cells (clamped) | One cell at a time, level order |
| `Query(k)` | Decay cells, return aggregate | One cell at a time |
| `UpdateAndQuery(k, δ)` | Update + return post-update aggregate | One cell at a time |
| `TryUpdate(k, δ, limit)` | Apply only if aggregate + δ ≤ limit | ALL L locks held simultaneously |
| `QueryDetailed(k)` | Per-level scores + cell indexes | One cell at a time |

## Key implementation details

- **Zero-allocation hot path**: Update (~111ns), Query (~117ns), UpdateAndQuery (~132ns) at L=4, M=100k on Apple Silicon. A `sync.Pool` of `*[]float64` buffers avoids allocations on the hot path.
- **Lock ordering**: Cells always acquired in ascending level order → deadlock-free. TryUpdate holds all L locks simultaneously.
- **NaN/Inf guard**: A NaN or Inf delta panics on the hot path — one poisoned cell corrupts every key hashing there.
- **Seed expansion for SipHash**: `Splitmix64` chain expands a single uint64 seed into the 128-bit SipHash key.
- **Clock backwards safety**: `Δt < 0` (clock going backwards) is treated as `Δt = 0`, never as score growth.
- **Validation at construction**: Rejects NaN params, `Lo ≥ Hi`, `Lo ≤ 0 ≤ Hi` violation, `Hi = +Inf` without effective decay, degenerate tuning inputs.

## Testing approach

The test suite is rigorous and spec-driven:

1. **Reference-model test** (`TestReferenceModel`): Random op sequences on a single key against a closed-form scalar reference, with a fake clock. Uses `pgregory.net/rapid` for property-based testing.
2. **Decay composition** (`TestDecayComposition`): Verifies `Apply(Apply(s,t1),t2) == Apply(s,t1+t2)` across all three built-in decays.
3. **TryUpdate atomicity** (`TestTryUpdateAtomicity`): 50 goroutines racing `TryUpdate(k, +1, 1000)` — exactly 1000 admitted, the property Query+Update can't guarantee.
4. **Determinism** (`TestSketchDeterminism`): Same seed + clock + ops = bit-identical output.
5. **Allocation enforcement** (`TestHotPathAllocations`): Update/Query/UpdateAndQuery must show 0 allocs/op.
6. **Goroutine leak detection**: `goleak.VerifyTestMain` in TestMain catches any Rotator that forgets to Close.
7. **Rotation tests**: Promotion order, warm-start (dual-write), goroutine wiring, Close idempotence.

## Four readings of the same mechanism

The same cell structure can be used for different purposes by varying the update pattern and decay:

1. **Decayed counter**: `+1` per event → per-key rate estimation (`Query × λ ≈ rate`)
2. **Saturating latch**: `+1`, clamp `Hi=1` → a Bloom filter that forgets (streaming dedup, novelty detection)
3. **Feedback score**: `±δ` on outcomes → FAIR-style fairness, abuse scores, circuit breakers
4. **Debt meter**: `+cost`, `Linear` decay → token-bucket rate limiting over unbounded keys

## Design decisions

- **Policy-free**: grudge stores and decays scores but makes no throttle/allow decisions. Callers interpret scores.
- **float64 cells**: Needed for pd-scale deltas (1e-5) that need the mantissa.
- **Per-sketch decay, not per-level**: One Decay function per sketch; nothing precludes per-level later.
- **Rotator reads from primary only**: Matches FAIR parity; cross-generation aggregation is unevaluated.
- **No key enumeration, no top-k**: Keys are hashed and discarded.
- **No serialization in v1**: Planned for vNext; the additive-update merge law is designed for convergent replica merge.

## Non-goals (v1)

- No decision policy (no throttle/allow, no Pi/Pd)
- No key enumeration or top-k
- No serialization, merge, or transport
- No windowing/quota math
- No multiplicative updates (would break the additive merge law)

## Milestones implemented

1. M1 — Core: cell, lazy decay, 3 Decays, 3 Aggregators, murmur3+KM hashing, Update/Query/UpdateAndQuery/QueryDetailed
2. M2 — TryUpdate with atomic conditional-consume
3. M3 — Rotator with background goroutine, grudgetest fakes
4. M4 — SuggestLevels tuning helper
5. M5 — SipHash for adversarial key spaces

## Performance

Apple Silicon, L=4, M=100,000:

| Operation | ns/op | allocs/op |
|---|---|---|
| Update | ~111 | 0 |
| Query | ~117 | 0 |
| UpdateAndQuery | ~132 | 0 |

Cell payload: ~6.4 MB per generation (L × M × cell size).
