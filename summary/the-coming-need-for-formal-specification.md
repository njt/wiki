---
url: https://benjamincongdon.me/blog/2025/12/12/The-Coming-Need-for-Formal-Specification/
title: "The Coming Need for Formal Specification"
author: Ben Congdon
date_fetched: 2026-05-14
date_published: 2025-12-12
topics:
  - software-engineering-craft
---

As AI increasingly handles code generation, software engineering's scarce resource shifts from writing code to specifying what code should do. Code itself is a poor map for system territory — modeling individual molecules tells you nothing about braking distance. Congdon engages with Martin Kleppmann's prediction that "AI will make formal verification go mainstream" and Hillel Wayne's observation that "you can probably fit every TLA+ expert in the world in a large schoolbus." The thesis: as generation costs plummet while review costs lag, systematic tooling for the mismatch becomes necessary. Congdon envisions a workflow starting with high-level English specs, decomposed into TLA+ models at multiple component specificity levels, with critical load-bearing components formally verified in Rocq and remaining components LLM-audited for spec conformance. He argues undergraduate CS programs should allocate curriculum to formal verification as students delegate implementation to AI.

## Structure

1. **The shifting bottleneck** — what happens when code generation is cheap but specification remains expensive
2. **Map–territory problem** — code as a poor map of system behavior
3. **Kleppmann's prediction** — AI formal verification going mainstream, seL4 case study (8,700 lines of C, 200,000 lines of Isabelle proofs)
4. **Current barriers** — TLA+ experts fit in a schoolbus, formal methods remain niche
5. **AI as bridge** — LLMs lowering the cost of formal specification the way they lowered the cost of code generation
6. **Wishcast future** — English specs → TLA+ decomposition → Rocq verification for critical paths → LLM auditing for the rest
7. **Education implications** — CS curricula shift from implementation to verification

## References

- Martin Kleppmann: https://martin.kleppmann.com/2025/12/08/ai-formal-verification.html
- Hillel Wayne: https://hillelwayne.com/post/why-dont-people-use-formal-methods/
- Map–territory relation: https://en.wikipedia.org/wiki/Map%E2%80%93territory_relation
- Congdon's related posts:
  - The Decline of the Software Drafter: https://benjamincongdon.me/blog/2025/12/08/The-Decline-of-the-Software-Drafter/
  - What I Look For in AI-Assisted PRs: https://benjamincongdon.me/blog/2025/12/10/What-I-Look-For-in-AI-Assisted-PRs/
