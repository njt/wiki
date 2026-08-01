---
url: https://emschwartz.me/unnecessary-optimization-in-rust-hamming-distances-simd-and-auto-vectorization/
title: "Unnecessary Optimization in Rust: Hamming Distances, SIMD, and Auto-Vectorization"
author: Evan Schwartz
date_fetched: 2026-08-01
date_published: 2024-12-22
site: emschwartz.me
---

# Unnecessary Optimization in Rust: Hamming Distances, SIMD, and Auto-Vectorization

Evan Schwartz, published 22 Dec 2024.

The author uses binary vector embeddings and Hamming distance in Scour, a service that scours noisy feeds for content matching user interests. He got "nerd sniped" into benchmarking Hamming distance implementations in Rust and published a new crate: hamming-bitwise-fast.

## Benchmark Setup

Contestants: naive for-loop, naive iterator, hamming, hamming_rs (x86/x86_64 only), simsimd (links a C library with platform-specific instructions), and hamming-bitwise-fast (23 SLOC, amenable to auto-vectorization).

Sizes tested: 512, 768, 1024, 2048 bits. Machines: 2023 MacBook Pro M2 Max, Linode dedicated CPU (2 vCPU/4 GB), Fly.io performance node (2 CPU/4 GB). Excluded: distances, stringzilla, triple_accel (they compute Hamming distance on strings, not bit-vectors).

## Results (nanoseconds)

MacBook Pro M2 Max: hamming-bitwise-fast won all sizes (4.6 / 6.5 / 4.6 / 7.5 ns). Notably, the naive for-loop and iterator took second/third (4.8–11.9 ns), beating hamming, simsimd, and hamming_rs (n/a — doesn't compile on ARM).

Linode: hamming-bitwise-fast won 512–1024 bits (9.3 / 12.3 / 16.0 ns); simsimd won 2048 bits (27.8 ns vs 28.8). Naive implementations were ~4–10× slower. hamming_rs: 24.5–70.7 ns.

Fly.io: hamming-bitwise-fast won 512–1024 bits (11.6 / 14.1 / 18.5 ns); simsimd won 2048 bits (32.7 vs 33.3 ns). Naive implementations 51.0–202.7 ns.

## Full Implementation

```rust
#[inline]
pub fn hamming_bitwise_fast(x: &[u8], y: &[u8]) -> u32 {
    assert_eq!(x.len(), y.len());

    // Process 8 bytes at a time using u64
    let mut distance = x
        .chunks_exact(8)
        .zip(y.chunks_exact(8))
        .map(|(x_chunk, y_chunk)| {
            let x_val = u64::from_ne_bytes(x_chunk.try_into().unwrap());
            let y_val = u64::from_ne_bytes(y_chunk.try_into().unwrap());
            (x_val ^ y_val).count_ones()
        })
        .sum::<u32>();

    if x.len() % 8 != 0 {
        distance += x
            .chunks_exact(8)
            .remainder()
            .iter()
            .zip(y.chunks_exact(8).remainder())
            .map(|(x_byte, y_byte)| (x_byte ^ y_byte).count_ones())
            .sum::<u32>();
    }

    distance
}
```

A u128/16-byte-chunk variant was 30% faster on 512/768-bit vectors on the MacBook but slower on all other benchmarks.

## Conclusion: Auto-Vectorization FTW

Auto-vectorization lets the compiler apply SIMD without hand-written intrinsics. The author quotes matklad's Can You Trust a Compiler to Optimize Your Code?:

"Writing SIMD by hand is really hard — you'll need to re-do the work for every different CPU architecture."

"Compilers are tools. They can reliably vectorize code if it is written in an amenable-to-vectorization form."

The recipe for triggering auto-vectorization, per matklad: express the algorithm in terms of processing chunks of elements, and within each chunk ensure no branching so all elements are processed identically. hamming-bitwise-fast does exactly this by XORing u64 chunks and using .count_ones().

Key takeaways / caveats:
- All implementations are so fast the differences rarely matter in practice.
- hamming-bitwise-fast beat simsimd (a C library with hand-written platform-specific SIMD) on most benchmarks — a win for trusting the compiler.
- Open questions: why did naive implementations place 2nd on the MacBook? Why was the u128 variant faster only on the MacBook for certain sizes?
- Fly.io doesn't guarantee SIMD support; relying on auto-vectorization makes that less of a concern.
