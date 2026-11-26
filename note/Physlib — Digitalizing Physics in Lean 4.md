# Physlib — Digitalizing Physics in Lean 4

Physlib is an open-source, community-driven effort to formalize theoretical physics in Lean 4, and its landing page's one genuinely novel idea is signed, verifiable contribution reports that give credit for reviewing, refactoring, maintenance, and documentation — not just formalization heroics.

---

The page is nearly bare: a title ("Digitalizing Physics in Lean 4"), a one-line mission ("An open-source, community project to formalize theoretical physics"), a link to contributions, and a link to Contribution Reports. But that last element repays attention.

> "Core work on Physlib by individuals can be published in signed, verifiable reports covering reviewing, formalization, foundations, refactoring, maintenance and documentation."

Formalization projects have a chronic incentive problem: proving a theorem is a publishable event; the years of reviewing, refactoring, and keeping the library coherent are not. Physlib's report model makes that work *an artifact* — signed and verifiable, presumably leaning on Lean's cryptographic and mathematical infrastructure for the verification half. It is credentialing built into the toolchain rather than bolted on by academia or GitHub stars.

It's also a natural fit for Lean specifically. A formal library is machine-checked, so "this person did real maintenance work" is closer to a checkable claim than it is in ordinary software — the substrate makes signed credit plausible where elsewhere it would be aspirational.

**Themes:** #concept #tool #project

**Take:** The physics formalization itself is the announcement; the report model is the idea worth stealing. Any project with machine-checkable work products — proofs, benchmarks, verified builds — could adopt signed verifiable reports for its unglamorous labour. Whether the Physlib community actually sustains the overhead of writing and signing reports is the open question the landing page can't answer.

This source strengthens [[Ship Your Interpreter (Basis)]] — another case of Lean making "verified" a property of the artifact itself, here extended from proofs to provenance of who did the work. It nuances [[Why Rocq Is Better Than Lean for Program Verification]] — Korkut's critique is about verification ergonomics, while Physlib shows where Lean's momentum actually comes from: community projects, credit infrastructure, and physics/math formalization rather than program verification. It also rhymes with [[Proving It Works]] — both treat "evidence of work done" as a first-class, mechanically checkable artifact rather than a narrative claim.

---
*Sources: [[raw/physlib-io]], [[summary/physlib-io]]*
*Last updated: 2026-11-26*
