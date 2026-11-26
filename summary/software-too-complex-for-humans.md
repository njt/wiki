---
url: https://anil.recoil.org/notes/software-too-complex-for-humans
title: "Was software just always too complex for humans?"
author: Anil Madhavapeddy
date_fetched: 2026-11-26
date_published: 2026-11 (late 2026)
topics:
  - specifications-as-the-product
  - software-engineering-craft
---

Anil Madhavapeddy (recoil.org, OCaml/Unikernel ecosystem) opens with the reported 17-million-line AI-assisted Lean proof of the Riemann hypothesis — machine-verified, human-ununderstood — as a lens on software engineering's long-standing gap between theory of computation and deployed reality. He argues decades of human-written code were always beyond our comprehension capacity, and AI is now making the gap catastrophic rather than merely latent.

His thesis: better models and harnesses alone will only produce "mountains of slop code." Two structural changes are required, together:

1. **Derive everything from specifications.** Speedrun formal specifications for all services — and ultimately rebuilt OS stacks and even redesigned instruction sets — then derive all new software from them. Unikernels with libOS linking disciplines and shared-memory interconnects are unusually amenable: few interfaces, whole-stack specialisation at link time. Machine-checked fixes re-derived from a spec also take humans out of the security patch loop, ending the asymmetry where attackers move instantly and defenders wait on maintainer triage.

2. **Engineer diversity deliberately.** Software's historical monoculture (reuse means one bug hits millions of hosts) should be replaced by "genetic" mechanisms so services aren't all infected simultaneously — valid, since a good spec admits multiple implementations. But diversity must extend beyond code to models and harnesses: a handful of gated frontier models generating all software is "out of the frying pan into an AI controlled fire." Specs can stay singular (like RFCs); implementations should mutate.

The third change is social: open source's human-review trust contracts are already straining under AI disclosure disputes. Humans remain needed to decide what specs contain (as with RFCs) and for new CS research: debugging diversity when no two nodes run identical code, forcing simplicity onto agent-maintained systems, and allocating blame when things fail.

The page also carries the HN discussion, where commenters largely contest the diagnosis: incentives (build-over-maintain, "acceptable risk" executive trade-offs) rather than human cognitive limits; Postel's law as the ossified jank of the internet; and scepticism that spec-writing is any easier than the original problem ("the spec is the hard part").
