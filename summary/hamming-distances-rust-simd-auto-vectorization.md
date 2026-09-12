---
url: https://emschwartz.me/unnecessary-optimization-in-rust-hamming-distances-simd-and-auto-vectorization/
title: "Unnecessary Optimization in Rust: Hamming Distances, SIMD, and Auto-Vectorization"
author: Evan Schwartz
date_fetched: 2026-08-01
date_published: 2024-12-22
topics:
  - software-engineering-craft
---

Evan Schwartz got "nerd sniped" into benchmarking Hamming distance
implementations in Rust while working on Scour, a service that uses binary
vector embeddings to match content against user interests. He published the
results alongside a new crate: hamming-bitwise-fast.

The benchmark pitted naive for-loops and iterators against several SIMD-aware
crates (hamming, hamming_rs, simsimd) across three machines: an M2 Max MacBook
Pro, a Linode dedicated CPU, and a Fly.io performance node. Vector sizes ranged
from 512 to 2048 bits.

hamming-bitwise-fast — just 23 lines of Rust that XORs u64 chunks and uses
`.count_ones()` — won nearly every category. On the MacBook it led at all sizes
(4.6–7.5 ns), with the naive implementations surprisingly placing second and
third. On the x86 cloud machines it won at 512–1024 bits, with simsimd (a C
library using hand-written platform-specific SIMD) edging it out only at 2048
bits.

The core insight is that auto-vectorization works. By expressing the algorithm
as chunked processing with no branching inside each chunk, the compiler can
reliably apply SIMD without hand-written intrinsics — portable across
architectures. This even beat simsimd's hand-tuned code on most benchmarks. The
article cites matklad's "Can You Trust a Compiler to Optimize Your Code?" for
the recipe and philosophy.

Caveats: all implementations are so fast the differences rarely matter in
practice. Open questions remain about why naive code performed so well on Apple
Silicon and why a u128-chunk variant only won on the MacBook at certain sizes.
