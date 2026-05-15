---
title: "Life at Low Reynolds Numbers"
url: https://www.damtp.cam.ac.uk/user/tong/fluids/lowreynolds.pdf
date_fetched: 2026-05-14
fetched_via: "web.archive.org PDF (https://web.archive.org/web/20251013061917/https://www.damtp.cam.ac.uk/user/tong/fluids/lowreynolds.pdf)"
section: "Random"
---

# Life at Low Reynolds Numbers
By E.M. Purcell, Lyman Laboratory, Harvard University, Cambridge, Massachusetts 02138.

Published: American Journal of Physics, Vol. 45, No. 1, January 1977. Copyright 1977 American Association of Physics Teachers.

Editor's note: This is a reprint (slightly edited) of a paper of the same title that appeared in the book *Physics and Our World: A Symposium in Honor of Victor F. Weisskopf*, published by the American Institute of Physics (1976). The personal tone of the original talk has been preserved in the paper, which was itself a slightly edited transcript of a tape.

## Summary

This is an after-dinner talk about what life is like for organisms swimming at very low Reynolds numbers -- the world of bacteria and microorganisms where viscosity completely dominates over inertia.

### The Reynolds Number

The Reynolds number describes the ratio of inertial forces to viscous forces in fluid dynamics:

R = (inertial forces / viscous forces) = (avp / n) = (av / v)

For water: R ~ 10^4 cm (where a = size, v = velocity). For a human swimming: R ~ 10^4 (large). For a swimming bacterium: R ~ 10^-4 or 10^-5 (extremely small). For these tiny animals, inertia is totally irrelevant.

### What Low Reynolds Number Means

At low Reynolds number, if a man were in proportionally the same situation as his sperm:
- He would be swimming in a fluid like molasses
- His maximum speed would be about a centimetre per minute
- Coasting distance after stopping: about 0.1 Angstroms
- Coasting time: about 0.6 microseconds

"This makes it clear what low Reynolds number means. Inertia plays no role whatsoever. If you are at very low Reynolds number, what you are doing at the moment is entirely determined by the forces that are exerted on you at that moment, and by nothing in the past."

### The Scallop Theorem

"There is a very funny thing about motion at low Reynolds numbers, which is the following. One special kind of swimming motion is what I call a reciprocal motion. That is to say, I change my body into a certain shape and then I go back to the original shape by going through the sequence in reverse."

At low Reynolds number, everything reverses in time. If you take the Navier-Stokes equation and throw away the inertia terms, all you have left is VP = n * V^2 * v, where p is the pressure. If you try to swim by a reciprocal motion, it can't go anywhere. Fast or slow, it exactly retraces its trajectory and it's back where it started.

"A good example of that is a scallop. You know, a scallop opens its shell slowly and closes its shell fast, squirting out water. The moral of this is that the scallop at low Reynolds number is no good. It can't swim because it only has one hinge, and if you have only one degree of freedom in configuration space, you are bound to make a reciprocal motion."

The simplest animal that can swim that way is an animal with two hinges (like a boat with a rudder at both front and back, and nothing else). Its configuration space is two-dimensional, and the animal goes around a loop -- "Time doesn't matter. The pattern of motion is the same, whether slow or fast, whether forward or backwards in time."

### How Bacteria Actually Swim

Real microorganisms use non-reciprocal strategies:

**E. coli:** About 2 micrometers long, swims by rotating flagella. Howard Berg at Harvard built a tracker that could follow individual bacteria -- a remarkable piece of machinery with x, y, z coordinates that could track a single bacterium while it was behaving normally. E. coli runs for about half a second in one direction at about 20-40 micrometers/second, then stops and goes off in some other direction ("tumbles").

"E. coli on the keel is about 2 cm long. The tail is about 1 cm long. The tail is the part that we are interested in. That's the flagellum. Some E. coli cells have them coming out the sides and they may have several, but when they have several they tend to bundle together. Some cells are nonstoked and don't have flagella. They live perfectly well, so swimming is not an absolute necessity."

The flagellum is only about 130 Angstroms in diameter. It is much thinner than the cilium, "which is another very important kind of propulsive machinery."

**The corkscrew mechanism:** Classic work by Sir Geoffrey Taylor, "the famous fluid dynamicist of Cambridge." He built a model: a cylindrical body with a helical tail driven by a rubber band motor inside the body. He tested it in glycerine. "In order to make the tail he hadn't just done the simple thing of having a turning corkscrew, because at that time nearly everyone had persuaded themselves that the tail doesn't rotate, it waves."

Berg proved that E. coli flagella actually rotate (rather than wave). Silverman and Simon at UC San Diego confirmed this with a mutant strain of E. coli that doesn't make flagella at all but only makes "something called the proximal hook to which the flagella would have been attached." When antibodies caused these hooks to glue together and one bacterium was stuck to a microscope slide, the whole body rotated at constant angular velocity.

### Propulsion Efficiency

The propulsion efficiency for these organisms is "more or less proportional to the square of the off-diagonal element of the matrix." For the models tested, that factor is more like 1.5 (rather than the theoretical limit of 2). "That's a factor minus 1 that counts, that's very bad for efficiency."

The propulsion matrix must be not symmetric ("the thing would not rotate at all"), and the two spirals must be of opposite handedness.

### Diffusion vs. Swimming

"I've now introduced the word diffusion. Diffusion is important because of another very peculiar feature of the world at low Reynolds number, and that is, stirring isn't any good."

At low Reynolds number, you can't shake off your environment. If you move, you take it along; it only gradually falls behind. The time for transporting anything a distance l by stirring is l/v, while for diffusion it's l^2/D.

The Sherwood number S = lv/D describes the ratio of transport by stirring versus diffusion. For bacteria: S ~ 10^-2, meaning swimming doesn't help with nutrient uptake. "The bug's problem is not its energy supply; its problem is its environment."

"It might as well wait for stuff to diffuse in or out. The transport of wastes away from the animal and food to the animal is entirely controlled locally by diffusion. You can thrash around a lot, but the fellow who just sits there quietly waiting for stuff to diffuse will collect just as much."

Why swim then? To increase food supply by 10%, a bacterium would need to move at a speed of 700 micrometers/second, which is 20 times as fast as it can swim. Swimming is about getting to a new location, not stirring up the immediate environment.

### Chemotaxis

Bacteria navigate by chemotaxis -- detecting concentration gradients. Berg's experiments with E. coli showed they "gradually work their way upstream" toward nutrients. The algorithm is simple: "if things are getting better, don't stop so soon" (run longer between tumbles). If things are getting worse, change direction sooner. "That's a very simple rule for working your way to where things are better."

There's a minimum "sort of bedrock length" below which further shortening of runs doesn't help, because diffusion around the cell dominates.

### On Osborne Reynolds

"He was a professor of engineering, actually. He was the one who not only invented Reynolds number, but he was also the one who showed what turbulence amounts to and that there is instability in flow, and all that. He is also the one who solved the problem of how you lubricate a bearing, which is a very visible problem that I recommend to anyone who hasn't looked into it."

Reynolds also published "a very long paper on the details of the subnumerical universe, and he had a complete theory which involved small particles of diameter 10^-18 cm."

### Closing (from another page in the journal)

The final page includes a Picasso quote about teaching: "So how do you go about teaching them something new? By mixing what they know with what they don't know. Then, when they see vaguely in their fog something they think, 'Ah, I know that.' And then it's just one more step to, 'Ah, I know the whole thing.' And their mind thrusts forward into the unknown and they begin to recognize what they didn't know before and they increase their powers of understanding."

-- Picasso, in *Life with Picasso* by Francoise Gilot and Carlton Lake (Nelson, London, 1965), p. 66.
