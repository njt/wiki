# Life at Low Reynolds Numbers

E.M. Purcell's classic 1977 after-dinner talk (American Journal of Physics, Vol. 45, No. 1) about what life is like for organisms so small that viscosity dominates over inertia. A Nobel laureate explaining physics to fellow physicists with hand-drawn diagrams and infectious enthusiasm.

---

## Key Quotes

> "If you are at very low Reynolds number, what you are doing at the moment is entirely determined by the forces that are exerted on you at that moment, and by nothing in the past."

> "A good example of that is a scallop. You know, a scallop opens its shell slowly and closes its shell fast, squirting out water. The moral of this is that the scallop at low Reynolds number is no good."

> "You can thrash around a lot, but the fellow who just sits there quietly waiting for stuff to diffuse will collect just as much."

> "Time doesn't matter. The pattern of motion is the same, whether slow or fast, whether forward or backwards in time."

> "The bug's problem is not its energy supply; its problem is its environment."

## Key Themes

#physics #biology #fluid-dynamics #science-communication #bacteria

The Reynolds number is the ratio of inertial forces to viscous forces in a fluid. At human scale (high Reynolds number ~10^4), inertia matters -- push off a wall and you coast. At bacterial scale (low Reynolds number ~10^-5), inertia is essentially zero. When you stop pushing, you stop instantly. The coasting distance of a bacterium after it stops swimming is about 0.1 Angstroms -- less than the diameter of an atom. There is no coasting.

This leads to the **scallop theorem**: any symmetric, back-and-forth (reciprocal) motion produces zero net displacement at low Reynolds numbers. Open, close, open, close -- you end up exactly where you started, regardless of speed. The physics is time-reversible: the Navier-Stokes equation without inertia terms doesn't care which direction time runs. To swim, you need at least two degrees of freedom and a non-reciprocal stroke pattern.

Bacteria solve this with **rotating helical flagella** -- a corkscrew, not a paddle. Sir Geoffrey Taylor built the first physical model (cylindrical body, helical tail driven by rubber band motor in glycerine). Howard Berg at Harvard then built a remarkable tracker that could follow individual E. coli bacteria, revealing their run-and-tumble navigation strategy: swim straight for about half a second, then randomly reorient and go again.

The surprise is **why bacteria swim at all**. At these scales, swimming doesn't help with feeding. The Sherwood number (ratio of stirring transport to diffusion transport) for bacteria is about 10^-2 -- diffusion completely dominates. A bacterium that sat perfectly still would collect nutrients just as efficiently as one that swam frantically. Swimming is about getting to a new location where concentrations are different, not about stirring up the immediate environment.

**Chemotaxis** is how bacteria use this: they detect whether conditions are improving or worsening and adjust their tumble frequency. If things are getting better, run longer. If things are getting worse, tumble sooner. A simple algorithm that doesn't require memory of absolute concentrations -- only a sense of gradient.

The talk ends with a digression on Osborne Reynolds himself -- not just the Reynolds number, but also the man who explained turbulence, solved bearing lubrication, and published a long paper theorizing about sub-numerical particles of diameter 10^-18 cm.

## Critical Analysis

This is one of the great pieces of physics communication. It's been reprinted, cited, and taught for nearly fifty years because it achieves the rare combination of scientific rigor and complete accessibility. Purcell uses no equations that a physics undergraduate couldn't follow (and most of the paper uses no equations at all), yet the insights are deep and non-obvious.

The "metre per week" thought experiment -- imagining a man swimming in molasses with proportionally the same Reynolds number as a bacterium -- makes the physics viscerally understandable. The scallop theorem, which sounds like a mathematical abstraction, becomes obvious once you hear it explained: of course a symmetric motion can't go anywhere when time-reversal doesn't change the physics.

The most surprising result is the futility of swimming for feeding. Everything in biology has a "just-so story" about adaptive advantage, and you'd assume swimming helps bacteria eat. It doesn't -- diffusion handles nutrient transport regardless. Swimming is purely about getting to a better neighborhood. This is the kind of counterintuitive result that only comes from doing the physics properly rather than reasoning from analogy to human experience.

The paper is also a beautiful example of what makes great science communication: Purcell doesn't do anything technically difficult -- he just explains familiar physics in an unfamiliar context with carefully chosen imagery. The excellence is in the selection and presentation, not in novel research. It's the scientific equivalent of [[The Mundanity of Excellence]] -- exceptional results from ordinary methods applied with extraordinary care.

The closing Picasso quote on teaching ("by mixing what they know with what they don't know") is a perfect meta-commentary on what Purcell just did.

---
*Sources: [[raw/life-at-low-reynolds-numbers]]*
*Last updated: 2026-05-14*
