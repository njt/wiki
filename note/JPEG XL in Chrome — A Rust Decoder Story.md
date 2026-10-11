# JPEG XL in Chrome — A Rust Decoder Story

Chrome 155 ships JPEG XL decoding, and the Chrome team's blog post is less a format announcement than a security-engineering essay: the decoder is `jxl-rs`, a pure Rust reimplementation chosen to eliminate memory-safety bugs at the source rather than trusting the sandbox to contain them, with a SIMD abstraction layer (`jxl_simd`) built to keep performance parity with the C++ reference decoder without blanket `unsafe`.

---

## Key Quotes

> "Sandboxing is a secondary layer of defense. To eliminate these security risks at the source, we have integrated `jxl-rs`, a pure Rust implementation of the JPEG XL decoder."

Image decoders run in the renderer process against untrusted network binary, and C++ decoders historically breed out-of-bounds reads, heap overflows, and use-after-frees. The quote is a crisp statement of defense-in-depth ordering: containment is not a substitute for a safe substrate. Commentary: the same layered logic recurs throughout agent sandboxing — a VM around the agent does not excuse unsafe tooling inside it.

> "A memory-safe decoder that is approximately as fast as the best non-memory-safe alternative is a much more obvious choice than a choice with a significant performance compromise."

The team refuses the old trade-off framing. Safety only wins adoption when it costs nothing. Commentary: this is the exact condition that Rust's ecosystem has been working toward — the safety argument alone was never enough for performance-critical code.

> "`target_feature_11` Rust feature had to be stabilized, which allowed the use of SIMD instructions without requiring `unsafe` code."

Unsafe was confined to "a small number of highly-vetted locations" via `jxl_simd`, an abstraction layer inspired by Highway. Commentary: the pattern is safety at the type level with carefully fenced unsafe islands — not the abolition of unsafe, but its quarantine.

> "We verified the `jxl-rs` implementation with various state-of-the-art techniques, including fuzzing and AI review of the code, and have not found any memory safety bugs throughout the entire implementation history."

Fuzzing plus AI code review as the verification pair. Commentary: notable that "AI review of the code" now sits alongside fuzzing in a mainstream security-verification toolkit — a Chrome production decoder trusted partly on an AI reviewer's pass.

## Key Themes

- #concept Memory safety at the source over containment: sandboxing as second line, language choice as first.
- #tool `jxl-rs` and `jxl_simd`: a Rust decoder achieving performance parity with libjxl by quarantining unsafe SIMD behind one abstraction layer.
- #concept File formats as attack surface: codecs process hostile untrusted binary in the renderer — the same reason [[Agent Substrate]]-style isolation matters elsewhere.
- #concept Standards by developer feedback: JPEG XL shipped via the Interop Project, not vendor fiat.

## Analysis

The post's most interesting move is how it resolves the safety-versus-speed dilemma that kept Rust out of performance-critical browser code for years. Rather than accepting a slower safe decoder, Google waited for — and arguably helped force — the stabilization of `target_feature_11`, then built a SIMD abstraction so that the unsafe footprint shrinks to vettable islands. The claim of zero memory-safety bugs across the whole implementation history is unverifiable from the outside but consistent with Rust's track record, and the honest framing ("approximately as fast") avoids overclaiming parity with a decade-tuned libjxl.

The secondary story is process: JPEG XL's return to Chrome (it was famously removed in 2023) came through the Interop Process, a case study in web-platform governance responding to sustained developer pressure. Chrome still hedges with "try both AVIF and JPEG XL", which reads as an admission that no single format wins — the web's image-format standards war ends in a portfolio, not a victor.

What the post doesn't say: JPEG XL's codec advantage claims (30–50% better than JPEG) are exactly the kind of received wisdom the encoding world has learned to distrust — encoder and settings dominate format, as arch1t3cht's video guide demonstrates. The durability claim will be tested by ecosystem adoption, not the announcement.

## Related Pages

- [[Unnecessary Optimization in Rust]] — complicates this post's SIMD story: Schwartz argues compilers can auto-vectorize well enough that handwritten SIMD (even safe, feature-gated SIMD) is often unnecessary; `jxl_simd` is the counterexample where a codec's inner loops justify the abstraction-layer investment.
- [[What You NEED to Know Before Touching a Video File]] — strengthens the format-versus-tooling theme: arch1t3cht's guide shows encoder settings dominate codec choice, a caution against reading Chrome's "30–50% better compression" headline at face value.
- [[A Deep Dive on Agent Sandboxes]] — nuances the sandboxing hierarchy: the same "containment is the second line, not the first" logic Chrome applies to decoders is the founding argument for sandboxing agents themselves.
- [[Agent Substrate]] — Google's sibling engineering culture at work: gVisor/micro-VM isolation for agents as the same rule-of-two, defense-in-depth mindset applied to a different attack surface.

---
*Sources: [[raw/jpeg-xl-in-chrome]], [[summary/jpeg-xl-in-chrome]]*
*Last updated: 2026-10-11*
