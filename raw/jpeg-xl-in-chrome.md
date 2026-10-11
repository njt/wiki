---
url: https://developer.chrome.com/blog/jpeg-xl-in-chrome
date_fetched: 2026-10-11
---

Published: October 6, 2026

We're excited to announce that Chrome is shipping decoding support for the JPEG
XL (`.jxl`) image format starting from Chrome 155. JPEG XL is a next-generation
image format designed to meet the needs of modern web developers and
photographers. It offers 30-50% better compression than JPEG, lossless
compression, built-in HDR support, lossless JPEG transcoding, and more.

In general, we recommend trying both AVIF and JPEG XL to get the best results. We expect that JPEG XL is most helpful for high-fidelity or lossless compression, especially of photographic images or in cases in which fine-grained progressive decoding is preferred.

In this post, we share why we brought JPEG XL to Chrome, how we used Rust to ensure memory safety first, the extensive performance work that makes it fast, and what the journey tells us about developer feedback and the web standards ecosystem.

## Safety first: Reimplementing the decoder in Rust (`jxl-rs`)

Image decoders are one of the most critical and targeted attack surfaces in any modern web browser. They process complex, untrusted binary structures directly from the network and run inside the renderer process. Historically, decoders written in memory-unsafe languages like C++ have been prone to vulnerabilities such as out-of-bounds reads, heap overflows, and use-after-free bugs.

Our security model relies on sandboxing and defense-in-depth, guided by the
rule of
two.
However, sandboxing is a secondary layer of defense. To eliminate these security
risks at the source, we have integrated `jxl-rs`, a pure Rust implementation of
the JPEG XL decoder.

## Design for speed, without compromising safety

Memory safety is crucial, but a memory-safe decoder that is approximately as fast as the best non-memory-safe alternative is a much more obvious choice than a choice with a significant performance compromise.

A fundamental part of the performance of modern codecs is making full use of the
SIMD hardware available on modern devices. To do so safely,
`target_feature_11` Rust
feature had to be stabilized, which allowed the use of SIMD instructions without
requiring `unsafe` code.

The next step was to build a SIMD abstraction layer (`jxl_simd`), inspired by
the C++ Highway library (itself originally
developed for `libjxl`, the C++ reference implementation of JPEG XL). Together,
those developments allowed writing a multi-platform library that doesn't
compromise on SIMD performance optimizations, while restricting unsafe
operations to a small number of highly-vetted locations.

Performance optimizations in `jxl-rs` build on those in `libjxl`. This includes
a generic processing pipeline for steps crossing region borders, while
minimizing data copies to maximize hardware performance. We've been tracking the
performance of the Rust reimplementation across different hardware platforms on
the jxl-rs performance dashboard.

We verified the `jxl-rs` implementation with various state-of-the-art
techniques, including fuzzing and AI review of the code, and have not found any
memory safety bugs throughout the entire implementation history, providing yet
another validation of the huge improvements that Rust brings to memory safety.

## Developer feedback and the Interop Project

The Chrome team considers web developer feedback from a wide range of channels, such as bugs, surveys, the Developer Signals Project, and the Interop Project. Our decision to ship JPEG XL was based on consistent feedback and requests from web developers, most visible in the Interop Process, where it was a popular proposal in 2026 and several years prior.

To ensure the format is interoperable across browsers, we have participated in the Interop 2026 JPEG XL Investigation to ensure there is test coverage for all of JPEG XL's features in browsers, and that those tests pass in Chrome.

## Try it out

With JPEG XL officially landing in Chrome, the web becomes faster, richer, and
safer. We encourage developers, content creators, and platform owners to start
using `.jxl` images and animations in their pipelines.

Try it out, file bugs, and help us continue building a faster and safer web for everyone.

## Acknowledgements

We'd like to thank all the people who contributed to `jxl-rs` or its integration
in Chrome, and especially Helmut Januschka for the substantial contributions
both to the Chrome integration and `jxl-rs`, and Martin Bruse, Zoltan Szabadka,
Sami Boukortt and Wonwoo Choi for their substantial contributions to `jxl-rs`
itself.
