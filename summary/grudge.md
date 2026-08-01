---
url: https://github.com/satmihir/grudge
title: "grudge — A constant-memory sketch that holds grudges and forgives"
author: Mihir Sathe (satmihir)
date_fetched: 2026-07-11
date_published: 2025
---

grudge is a Go library implementing a constant-memory decaying-score sketch. It maps an unbounded key space to scalar scores that update with feedback and decay toward zero autonomously — the behavioral-score analogue of a count-min sketch. Extracted from the FAIR project (Stochastic Fair BLUE-based fairness throttling).

The core structure is a Sketch: L levels × M cells, each cell holding a float64 score and a last-updated timestamp behind a mutex. Every key maps to exactly one cell per level via Kirsch-Mitzenmacher hash derivation, so all operations touch only L cells regardless of key cardinality. Scores decay lazily on access — idle sketches do zero background work. A Rotator manages multiple generations under different hash seeds, reading from the oldest and writing to all, with periodic rotation to bound false-positive lifetime.

Three defining properties set it apart from simpler sketches. **Collision shielding**: with Min aggregation, a key's estimate is exact whenever at least one of its L cells is free of colliding traffic; error is one-sided (Query ≥ true score). **Lazy decay**: decay is computed only on access and must compose across time subdivision. **Rotation**: worst-case false positives last at most Generations × Period.

The sketch supports five operations — Update, Query, UpdateAndQuery, TryUpdate (atomic conditional-consume holding all L locks), and QueryDetailed. The hot path is zero-allocation (≈111–132ns at L=4, M=100k on Apple Silicon). Guardrails include NaN/Inf panics, clock-backwards safety, and validation at construction.

The same mechanism supports four distinct readings by varying the update pattern and decay: a decayed counter for rate estimation, a saturating latch as a forgetful Bloom filter, a feedback score for fairness/abuse scoring, and a debt meter for token-bucket rate limiting over unbounded keys.

The test suite is notably rigorous — property-based reference-model tests, decay-composition verification, TryUpdate atomicity under 50 racing goroutines, determinism checks, allocation enforcement, and goroutine leak detection.

V1 is intentionally policy-free (no throttle/allow decisions), with no key enumeration, serialization, or top-k support. Serialization with convergent replica merge is planned for a future version.
