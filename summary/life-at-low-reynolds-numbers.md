---
url: https://davidtong.org/pdfs/teaching/fluid-mechanics/lowreynolds.pdf
title: "Life at low Reynolds number"
author: E. M. Purcell
date_fetched: 2026-08-09
date_published: 1977-01
topics:
  - ideas-and-culture
---

Purcell takes his audience into the world of very low Reynolds numbers — the
world inhabited by microorganisms. At micron scales, Reynolds numbers are
~10⁻⁴–10⁻⁵; inertia is irrelevant, and motion is determined entirely by forces
exerted at each moment. A swimming bacterium that stops pushing coasts only ~0.1 Å
and halts in under a microsecond.

The central insight is the **scallop theorem**: at low Reynolds number, any
reciprocal motion — deforming a body into a shape and then returning through the
same sequence in reverse — produces zero net displacement. A scallop, with only
one hinge, cannot swim in this regime. To move, an organism needs a non-reciprocal
cycle, which requires at least two degrees of freedom in configuration space.

Real microorganisms solve this with flexible oars (which bend differently on each
half-stroke) or corkscrew drives. Purcell shows that _E. coli_ swims by
continuously rotating its helical flagellum — a genuine rotary motor — confirmed by
Berg's tracking experiments and Silverman & Simon's tethered-cell assays. The
propulsion matrix for a helix is symmetric (only three independent constants), and
efficiency is low — roughly 1%. But that barely matters: the energy budget is only
~0.5 W/kg, a small fraction of metabolism. "They're driving a Datsun in Saudi
Arabia."

The real constraint at low Reynolds number is **diffusion**. Stirring is useless
locally — the "stirring number" (Sherwood number) _lv/D_ is ~10⁻² at these scales,
so transport of nutrients and wastes is entirely diffusion-controlled. A bacterium
doesn't swim to scoop up more food (that would require speeds 20× faster than it
can achieve); it swims to find greener pastures. To outrun diffusion and
meaningfully sample its environment, it must travel a distance of at least _D/v_
≈ 30 µm — which is exactly the distance Berg observed in _E. coli_ tracks.

Berg also discovered the chemotaxis algorithm: when things are getting better,
don't stop so soon. When things are getting worse, path lengths don't shorten —
Purcell speculates that's because there's a bedrock length set by the
diffusion-outrunning distance, below which sampling makes no sense.

Originally a talk honoring Victor Weisskopf, the paper is a model of physical
storytelling — playful, personal, and built from elementary physics applied to an
unfamiliar domain. It shows that at low Reynolds number, what matters isn't
propulsion efficiency or energy, but information: how an organism navigates a
world it cannot stir.
