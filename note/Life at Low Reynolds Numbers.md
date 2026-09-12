# Life at Low Reynolds Numbers

E.M. Purcell's classic 1977 after-dinner talk (American Journal of Physics, Vol. 45, No. 1) about what life is like for organisms so small that viscosity dominates over inertia. Delivered as a talk in honor of Victor Weisskopf (the "Viki" addressed throughout), then transcribed from tape with the personal tone preserved. Purcell was a Nobel laureate (NMR, 1952) who never lost the amateur's enthusiasm for applying elementary physics to unfamiliar domains.

---

## Key Quotes

> "If you are at very low Reynolds number, what you are doing at the moment is entirely determined by the forces that are exerted on you at that moment, and by nothing in the past."

Purcell adds a footnote: "In that world, Aristotle's mechanics is correct!" At low Re, F = mv, not F = ma. Velocity is proportional to force, not acceleration. The physics of the microscopic world vindicates Aristotle against Newton.

> "A good example of that is a scallop. You know, a scallop opens its shell slowly and closes its shell fast, squirting out water. The moral of this is that the scallop at low Reynolds number is no good."

This is the **scallop theorem**: any reciprocal motion (symmetric back-and-forth) produces zero net displacement when inertia is absent. One degree of freedom means you can only do reciprocal motion. To swim, you need at least two hinges — a loop in configuration space, not a line.

> "You can thrash around a lot, but the fellow who just sits there quietly waiting for stuff to diffuse will collect just as much."

The most counterintuitive finding in the paper. Everything in biology has a just-so story about adaptive advantage. Swimming doesn't help bacteria eat. Diffusion handles all local transport. Swimming is purely for relocation.

> "Time doesn't matter. The pattern of motion is the same, whether slow or fast, whether forward or backwards in time."

The Navier-Stokes equation without the inertia term is time-reversible. A film of low-Re swimming played backwards is physically indistinguishable from one played forwards. This is why reciprocal motion fails: the return stroke exactly undoes the forward stroke, regardless of speed.

> "The bug's problem is not its energy supply; its problem is its environment."

> "They're driving a Datsun in Saudi Arabia."

On propulsion efficiency being ~1%: it literally doesn't matter. The energy cost is 0.5 W/kg — a small fraction of bacterial metabolism. Nature didn't optimize for efficiency because it didn't need to.

## Key Themes

#physics #biology #fluid-dynamics #science-communication #bacteria #biophysics

The Reynolds number is the ratio of inertial forces to viscous forces in a fluid. At human scale (Re ~10^4), inertia matters — push off a wall and you coast. At bacterial scale (Re ~10^-5), inertia is essentially zero. When you stop pushing, you stop instantly. The coasting distance after an E. coli stops swimming is about 0.1 Å — less than the diameter of an atom. There is no coasting.

Purcell's vivid scaling argument: the characteristic force n²/ρ. For water, that's 10^-4 dyn — the force needed to tow anything, large or small, at Re ~1. If you want a Reynolds-1 submarine, tow it with 10^-4 dyn. Conversely, for Earth's mantle (n ~10^21 poise), n²/ρ ~10^41 dyn, more than 10^9 times the gravitational force between Earth's hemispheres. The mantle's Re is unimaginably small.

This leads to the **scallop theorem**: any reciprocal (back-and-forth) motion produces zero net displacement at low Re. The Navier-Stokes equation without the inertia term is time-reversible — a film of low-Re swimming played backwards looks physically identical. To swim, you need at least two degrees of freedom, making a loop in configuration space. The simplest conceptual swimmer: a boat with a rudder at both ends, with coordinates θ₁ and θ₂ — "the animal is going around a loop in that configuration space, and that enables it to swim."

Nature's solutions: **rotating helical flagella** (the corkscrew — E. coli), **flexible oars** (cilia that bend differently on forward and return strokes), and **rolling cells** (two cells stuck together, rolling on one another). Sir Geoffrey Taylor built the first physical model (cylindrical body, helical tail, rubber-band motor in glycerine). He sheathed the turning helix in rubber tubing because "at that time nearly everyone had persuaded themselves that the tail doesn't rotate, it waves." Berg and Silverman/Simon later proved rotation definitively — the bacterium has an actual rotary motor, a "most remarkable piece of machinery."

The **propulsion matrix** is a 2×2 matrix relating force and torque to linear and angular velocity. At low Re everything is linear, so the matrix fully describes propulsion. Purcell proves it must be symmetric (C = B), so only three constants matter. The propulsive efficiency is proportional to (B)², which depends on the difference between perpendicular and parallel drag on a thin wire. For the flagellum: about 1%.

The surprise is **why bacteria swim at all**. The Sherwood number (stirring/diffusion transport ratio) for bacteria is ~10^-2 — diffusion completely dominates local transport. A bacterium sitting still collects nutrients just as efficiently as one swimming frantically. "The bug can collect, by diffusion through the surrounding medium, enough energetic molecules to keep moving when the concentration of those molecules is 10^-9 M." Energy cost: 2×10^-8 ergs/sec, 0.5 W/kg — a tiny fraction of metabolism.

Swimming is about **outrunning diffusion** to find greener pastures. To matter, you must swim farther than D/v ~30 µm — and that's exactly the run length of E. coli. If you don't swim that far, "you haven't gone anywhere."

**Chemotaxis** is the navigation algorithm: if conditions are improving, don't stop so soon (runs get longer). If conditions are worsening, tumble sooner. Purcell speculates there's a "bedrock length" below which shorter runs are pointless — diffusion already samples the local environment. No memory of absolute concentrations needed, only a gradient sense.

The talk ends with a digression on Osborne Reynolds himself: invented the Reynolds number, explained turbulence and flow instability, solved bearing lubrication ("a very subtle problem that I recommend to anyone who hasn't looked into it"), and late in life published a long paper theorizing about sub-mechanical particles of diameter 10^-18 cm — "it gets very nutty from there on."

The paper closes with a Picasso quote on teaching: "by mixing what they know with what they don't know" — a perfect meta-commentary on what Purcell just did.

## Critical Analysis

This is one of the great pieces of physics communication, and it's worth understanding why it works so well when most "distinguished lectures" are instantly forgotten.

**The talk succeeds because Purcell actually does the physics, not just popularize it.** He derives the n²/ρ force scale, proves the propulsion matrix must be symmetric, calculates energy budgets, works out the diffusion/stirring crossover. None of this is mathematically hard — it's undergrad-level fluid mechanics — but ninety-nine out of a hundred speakers would skip the derivations and just describe the results. Purcell trusts his audience to follow the reasoning, and that trust is what makes the talk respect-worthy rather than condescending.

**The corn syrup demo is the ideal of scientific demonstration.** He projects a tank of corn syrup (50 poise, 5000× water) using an overhead projector turned on its side, drops in model helices, and lets the audience watch them "swim." The material is cheap, edible ("you can just lick the experimental material off your fingers"), and the phenomenon is *slow enough to watch*. No electronics, no data acquisition — just watching physics happen. This is the opposite of a PowerPoint talk.

**The most important sentence might be the opening one:** "This is a talk that I would not, I'm afraid, have the nerve to give under any other circumstances." The talk was for Victor Weisskopf's Festschrift — an occasion that gave Purcell permission to be playful. The institutional structure of the Festschrift created space for a kind of science communication that the regular seminar format would have crushed. A reminder that scientific culture depends on its rituals as much as its methods.

**The Datsun in Saudi Arabia line is the perfect throwaway.** After working through propulsion matrices and measuring efficiency at ~1%, Purcell realizes it doesn't matter and drops a one-liner that makes the whole calculation feel like a shaggy-dog story — which in a way it was. The willingness to follow a question to its answer and then shrug at the insignificance of that answer is something few academics can manage. Most would puff up the importance of their own measurement.

**The chemotaxis algorithm is a preview of computational biology before that was a field.** "If things are getting better, don't stop so soon" — that's a gradient-climbing algorithm stated in plain English. Berg's later work would formalize this into the signaling pathway of E. coli chemotaxis (one of the best-understood biological computing systems), but Purcell had the intuition in 1976.

**The talk's structure mirrors its thesis.** It moves from scaling arguments (Reynolds number) → constraints (scallop theorem) → mechanisms (rotary motors) → consequences (diffusion, chemotaxis) → historical footnote (Osborne Reynolds). Each step follows from the previous one with the inexorability of a low-Re flow — no inertia, no wasted motion, everything connected to what came before.

**What's missing:** Purcell barely mentions the molecular mechanism. He admits the motor is "very mysterious" and that "physicists aren't competent even to conjecture about it." In 1976 this was true. We now know the flagellar motor is a marvel of nanotechnology — a ~40-protein rotary engine driven by proton motive force, spinning at up to 100,000 rpm, with a universal joint, bushing, and reverse gear. The paper is more interesting, not less, for having been written before any of this was known. It shows how far you can get with just physical reasoning and no molecular biology.

**The Picasso quote at the end is the real conclusion.** Purcell didn't need to teach his audience fluid mechanics — they were physicists. He needed to teach them that physics they already knew applied to a domain they'd never thought about. He did it by "mixing what they know with what they don't know": Reynolds number is familiar, bacteria are unfamiliar, put them together and the unknown becomes legible. This is the method behind every successful piece of science communication.

---
*Sources: [[summary/life-at-low-reynolds-numbers]]*
*Last updated: 2026-05-18*
