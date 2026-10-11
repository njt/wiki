---
url: https://developer.chrome.com/blog/jpeg-xl-in-chrome
title: "JPEG XL in Chrome"
author: Chrome Team
date_fetched: 2026-10-11
date_published: 2026-10-06
topics:
  - security-and-sandboxing
  - databases-and-data
---

Chrome 155 ships native JPEG XL (`.jxl`) decoding: a next-generation image format claiming 30–50% better compression than JPEG, lossless compression, built-in HDR support, and lossless JPEG transcoding. Chrome's advice is pragmatic — try both AVIF and JPEG XL, with JPEG XL most helpful for high-fidelity or lossless photographic compression and fine-grained progressive decoding.

The bulk of the post is not about the format but about how it was integrated: the decoder is `jxl-rs`, a pure Rust reimplementation chosen because image decoders are among the most targeted attack surfaces in a browser. Decoders process complex, untrusted binary structures from the network inside the renderer process, and C++ decoders have historically been prone to out-of-bounds reads, heap overflows, and use-after-free bugs. Chrome's security model rests on sandboxing and the "rule of two", but the team explicitly calls sandboxing a *secondary* layer and eliminated the risk at the source by using a memory-safe language.

The interesting engineering tension is performance versus safety: "a memory-safe decoder that is approximately as fast as the best non-memory-safe alternative is a much more obvious choice". Two enablers made that possible — the stabilization of Rust's `target_feature_11`, which allows SIMD without `unsafe` code, and `jxl_simd`, a SIMD abstraction layer inspired by the C++ Highway library, restricting unsafe operations to a small number of highly-vetted locations. Verification combined fuzzing with AI review of the code, and the team reports zero memory-safety bugs across the entire implementation history.

Finally, the ship decision is framed as a response to developer feedback via the Interop Project: JPEG XL was a popular Interop 2026 proposal, and Chrome participated in the Interop 2026 JPEG XL Investigation to build cross-browser test coverage.
