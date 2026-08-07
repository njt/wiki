# Unnecessary Optimization in Rust

Evan Schwartz gets nerd-sniped into benchmarking six Hamming distance implementations for binary vector embeddings, publishes a 23-line crate that beats hand-tuned C SIMD libraries on most benchmarks, and emerges with a counterintuitive thesis: stop writing SIMD by hand and just write code the compiler can auto-vectorize.

---

## Key Quotes

> "Writing SIMD by hand is really hard — you'll need to re-do the work for every different CPU architecture."

Quoting matklad's *Can You Trust a Compiler to Optimize Your Code?*, Schwartz distills the central argument of the post: platform portability is a real engineering cost, and the compiler can shoulder it if you meet it halfway.

> "Compilers are tools. They can reliably vectorize code if it is written in an amenable-to-vectorization form."

The matklad recipe: process elements in fixed-size chunks, ensure no branching within each chunk so all elements are processed identically. Schwartz's `hamming-bitwise-fast` does exactly this — XOR `u64` chunks, call `.count_ones()`.

> "All implementations are so fast the differences rarely matter in practice."

The honesty of the post's title. After exhaustive benchmarking across three machines, the 23-line auto-vectorizable implementation wins most categories. But the margins are measured in nanoseconds. This is optimization as intellectual exercise, not production necessity.

## Key Themes

- **#pattern** The auto-vectorization recipe: chunked processing + branch-free inner loop + known-size integer operations = the compiler generates SIMD for you. No intrinsics, no per-platform code paths, no C FFI.
- **#tool** [`hamming-bitwise-fast`](https://crates.io/crates/hamming-bitwise-fast) — 23 SLOC of Rust that beat `simsimd` (C library with platform-specific SIMD) on most benchmarks. A case study in minimal code winning.
- **#concept** Compiler trust as engineering discipline. The post extends matklad's argument with empirical data: auto-vectorization beat hand-written SIMD on two of three platforms. The exception (2048-bit vectors on x86) was narrow and marginal.
- **#concept** Benchmarking as inquiry, not justification. Schwartz's open questions — why naive implementations placed 2nd on the M2 Max, why u128 chunks were faster only on one platform — model the right posture toward performance work: curiosity over certainty.

## Critical Analysis

**The real finding isn't about Hamming distance.** It's a case study in how the Rust compiler's LLVM backend has gotten good enough that the "obvious" optimization — hand-writing SIMD intrinsics or linking C libraries — is now often counterproductive. This is a pattern that repeats across domains: GPUs, database query planning, JIT compilation. The compiler is a moving target, and betting against it is increasingly a losing trade.

**The post gets its framing right in the title.** "Unnecessary Optimization" is both self-deprecating and precise. The benchmarks are exhaustive, the methodology is sound, and the conclusion is that it barely matters. That's the correct conclusion for 95% of performance work, and it's rare to see it stated so plainly.

**The "compiler trust" thesis has limits.** On the largest vectors (2048 bits) on x86, `simsimd` still won. The gap was small but consistent. The lesson isn't "never write SIMD" — it's "don't write SIMD until you've tried auto-vectorization and measured the gap." The post would be stronger if it acknowledged this is a pragmatic heuristic, not a universal law.

**The missing dimension: how representative is Hamming distance?** The algorithm is unusually SIMD-friendly — XOR, popcount, sum. Not every compute kernel is this regular. The auto-vectorization recipe works here because the problem fits the pattern perfectly. Generalizing "just trust the compiler" to irregular workloads would be a category error.

**Why this matters for the wiki.** This is a self-contained case study in a pattern that's orthogonal to the agentic-development focus of most pages here: the compiler as a tool that gets better over time, and the engineering discipline of writing code that *lets* it get better. The same dynamic plays out in LLM training, GPU kernel compilation, and database query optimization. The lesson — design for the optimizer, not against it — transfers broadly.

Schwartz's Hamming-distance deep-dive is a single-method optimization applied to a micro-benchmark; [[Performance Optimization Loop]] generalizes this into a full methodology: monitor in production, profile hotspots, benchmark baselines, apply small targeted changes, and validate each one independently. The same "curiosity over certainty" posture Schwartz models is what Gordon builds into a repeatable engineering practice.

---

*Sources: [[raw/hamming-distances-rust-simd-auto-vectorization]]*
*Last updated: 2026-08-07*
