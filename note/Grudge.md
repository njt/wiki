# Grudge

A constant-memory probabilistic sketch for Go that maps an unbounded key space to a decaying scalar score — it is to behavioral scores what a count-min sketch is to counts. Extracted from the FAIR fairness-throttling project, it provides the core data structure for rate limiters, abuse scoring, circuit breakers, cache admission, and fair schedulers in fixed memory (~6.4 MB for typical config) with ~120ns hot-path operations and zero background work.

#tool #project #data-structure #go #probabilistic

---

## Architecture

Grudge is a single Go package (~1,860 lines) structured as a **two-dimensional cell array with lazy time-decay semantics**. Three properties define the system:

- **Collision shielding**: L independent hash levels with min-aggregation. A key's score is exact whenever at least one of its L cells is untouched by colliding traffic. Error is one-sided: `Query(k) ≥ true score`, never under.
- **Lazy decay**: Scores fade toward zero, computed on access only. An idle sketch does literally zero work — no background goroutines, no periodic sweeps.
- **Rotation**: A `Rotator` keeps N generations under different hash seeds, reads from the oldest, writes to all, and periodically retires the oldest generation. Bounds any false positive's lifetime to `Generations × Period`.

### Core types

The entire system is built on three abstractions:

1. **`cell`** (`sketch.go:11-15`): `{score float64, lastUpdated int64, sync.Mutex}` — the atomic unit of state. 16 bytes of payload per cell. Each cell is independently locked; lock ordering (ascending level) guarantees deadlock freedom.

2. **`Decay`** interface (`decay.go:16-20`): `Apply(score float64, dtMillis int64) float64`. Must be monotone toward zero, never cross zero, and compose across time subdivision. Three built-ins: `Exponential(λ)` for proportional forgiveness, `Linear(rate)` for constant-rate drain (token-bucket refill), and `None` for identity.

3. **`Aggregator`** type (`aggregator.go:6`): `func([]float64) float64`. Collapses L per-level scores into one. `Min` is the recommended default — it provides the collision-shielding property since a key's estimate is dragged up only when *every* one of its cells is contamined. `Max` and `Mean` are also provided.

### Package layout

| Package | File | Role |
|---|---|---|
| `grudge` | `sketch.go` (263 lines) | Sketch struct, all five operations, lazy decay per cell, clamp logic |
| `grudge` | `decay.go` (114 lines) | Decay interface, Exponential/Linear/None implementations |
| `grudge` | `aggregator.go` (40 lines) | Min/Max/Mean aggregator functions |
| `grudge` | `config.go` (59 lines) | Config struct, validation (rejects NaN, degenerate ranges, Hi=+Inf without decay) |
| `grudge` | `rotator.go` (168 lines) | N-generation Rotator with background goroutine, RWMutex-guarded promotion |
| `grudge` | `tune.go` (56 lines) | `SuggestLevels`: solves `(1 − (1−1/M)^H)^L` for L given target false-positive probability |
| `grudge` | `hash.go` (42 lines) | Hasher/HasherFactory interfaces, Murmur3 + SipHash factories |
| `grudge/internal/hash` | `siphash.go` (89 lines) | Full SipHash-2-4 implementation (no external dependency) |
| `grudgetest` | `clock.go` (57 lines) | FakeClock, FakeTicker for deterministic test driving |

## Key techniques

### Kirsch–Mitzenmacher hash derivation

Rather than computing L independent hashes (which would multiply hash cost by L), grudge computes **one** 64-bit hash per key and derives L level indexes via `cell_index(l) = (h1 + l × h2) mod M`, where `h1` and `h2` are the low and high 32 bits of the hash (`sketch.go:234-237`). This is the same technique used in count-min sketch implementations.

The trade-off, documented explicitly in `hash.go:22-26`: "a single 64-bit collision collides at every level and bypasses min-aggregation shielding entirely." For attacker-controlled keys, the library provides `SipHash()` — a keyed PRF that resists collision manufacturing at roughly 2× the hash cost of murmur3.

### Lazy decay with clock-safety

Decay is applied on every cell access, before any other logic (`sketch.go:240-247`):

```go
func (s *Sketch) decayLocked(c *cell, now int64) {
    dt := now - c.lastUpdated
    if dt > 0 {
        c.score = s.decay.Apply(c.score, dt)
    }
    // dt <= 0 (including backwards clock) leaves score unchanged.
    c.lastUpdated = now
}
```

This is the core trick: scores evolve purely through lazy evaluation. There's no background goroutine, no periodic sweep, no timer. An idle sketch does nothing. The `Δt < 0` guard handles clock backwards safely — treating it as zero elapsed time, never as score growth.

### Decay composition as the correctness invariant

The key property that makes lazy decay correct is **time-subdivision composition**: `Apply(Apply(s, t1), t2) == Apply(s, t1+t2)`. This is verified via property-based testing (`decay_test.go:20-38`) with `pgregory.net/rapid` across all three built-in decays over ranges of ±1e6 scores and up to 200 seconds. Without this property, the lazy-evaluation strategy would diverge from continuous decay, and the planned merge law for replica convergence would not hold.

### Zero-allocation hot path via sync.Pool

Update, Query, and UpdateAndQuery must be allocation-free — enforced by `TestHotPathAllocations` using `testing.AllocsPerRun`. The trick: a `sync.Pool` of `*[]float64` buffers (`sketch.go:34, 81-84`) provides scratch space for the L-element score slices needed during aggregation. Each hot-path method gets a buffer from the pool, fills it, aggregates, and returns it. `QueryDetailed` is the exception — it allocates its returned slices by design.

### TryUpdate: atomic conditional-consume

`TryUpdate(key, delta, limit)` (`sketch.go:171-207`) is the only operation that holds **all L cell locks simultaneously**. It acquires them in ascending level order (deadlock-free because every operation uses the same order), decays all cells, computes the aggregate, and applies the delta only if `aggregate + delta ≤ limit`. This is what makes the token-bucket rate limiter pattern work — without it, a Query-then-Update would have a TOCTOU race.

The atomicity is verified by a sentinel test (`tryupdate_test.go:16-45`): 50 goroutines racing `TryUpdate(k, +1, 1000)` with `None` decay. Exactly 1000 admitted, never more, never fewer.

### Seed expansion via Splitmix64

The public API exposes hashing as `uint64` seed, but SipHash needs a 128-bit key. Rather than changing the API or using a separate key type, `SipHash()` (`hash.go:36-42`) expands the uint64 seed into two 64-bit keys via two iterations of Splitmix64 — a simple, non-cryptographic PRNG that maps well:

```go
k0 := hash.Splitmix64(seed)
k1 := hash.Splitmix64(k0)
```

### Rotation as dual-write warm-start

The Rotator (`rotator.go`) writes every update to **all** generations, not just the primary. When the oldest generation is retired and the next is promoted, it's already warm — no cold-start gap. Reads serve from the oldest generation only (the "primary"), providing a stable view. Promotion uses a write lock; all operations use a read lock (`sync.RWMutex`).

## Design decisions

**Policy-free by design.** Grudge stores and decays scores; it makes no throttle/allow/sample decisions. This is a deliberate separation of concerns — FAIR handles the policy (stochastic fairness), grudge provides the data structure. Callers interpret scores however they need.

**float64 cells, not float32.** Per the spec (§6), "Pd-scale deltas (1e-5) need the mantissa." The 8-byte cell score is the main memory cost. At L=4, M=100,000 with float64 + mutex + timestamp overhead, each cell is ~16 bytes → ~6.4 MB per generation. The spec leaves open the possibility of a packed `float32 score` + age in a single `uint64` if benchmarks justify it (vNext).

**Pluggable hashing with documented caveats.** Murmur3 is the default (fast, ~2× cheaper than SipHash), but the code explicitly documents its seed-independent collision families and the amplification hazard of Kirsch-Mitzenmacher derivation. The SipHash-2-4 implementation is in-tree (`internal/hash/siphash.go`, 89 lines) with zero external dependencies — a deliberate choice to keep the dependency footprint small.

**Additive update law preserved for future merge.** All updates are additive deltas (`score = clamp(score + delta)`). Multiplicative updates (true AIMD) are out of contract because they would break the additive merge law that vNext serialization depends on. This is a forward-looking constraint documented in the spec (§9).

**No key enumeration, no top-k.** Keys are hashed and discarded. This is both a performance choice (no secondary index to maintain) and a security choice (no way to enumerate what keys have been seen). If you need top-k, you build it on top.

## Comparison notes

**vs. count-min sketch**: A count-min sketch estimates frequency counts; grudge estimates decaying behavioral scores. Both use L×M cells and min-aggregation, but grudge adds the time dimension: scores evolve without explicit update, and the decay function is pluggable. Count-min sketches don't have a notion of forgiveness.

**vs. a Bloom filter**: A saturating latch configured with `Update(k, +1)`, `Hi=1`, `None` decay, and `Min` aggregation behaves like a Bloom filter that never forgets. Add `Exponential` decay and you get a Bloom filter that forgets — useful for streaming deduplication where old items should eventually become "new" again.

**vs. per-key state (hash maps)**: The standard approach to per-key rate limiting stores a map entry per key with a timestamp and counter. This works until key cardinality explodes — then you need eviction policies, background reaping, and the map becomes a scaling bottleneck. Grudge fixes memory at construction time and never allocates per key.

**vs. token bucket**: A conventional token bucket is per-key state. Grudge's debt-meter reading (`+cost`, `Linear` decay, `TryUpdate`) approximates a token bucket over unbounded keys in constant memory. Collisions only make it stricter — a fresh key reads as "full bucket," and a stable key is never falsely admitted because collisions only add positive bias.

## Four readings, one mechanism

The same cell structure serves four different purposes depending on how you configure updates and decay:

| Reading | Update | Decay | What you get |
|---|---|---|---|
| Decayed counter | `+1` per event | Exponential | `Query × λ ≈ rate` — per-key rate estimation, heavy-hitter detection |
| Saturating latch | `+1`, `Hi=1` | None or Exponential | A Bloom filter that forgets — streaming dedup, novelty detection |
| Feedback score | `±δ` on outcomes | Exponential | FAIR-style fairness scores, abuse scoring, circuit breakers |
| Debt meter | `+cost` | Linear | Approximate per-key token bucket — rate limiting over unbounded keys |

---

*Sources: [[raw/grudge]]*
*Last updated: 2026-07-11*
