# Software Too Complex for Humans (Madhavapeddy)

Anil Madhavapeddy's late-2026 polemic argues that software was always beyond human comprehension — the Riemann hypothesis cracked by 17M lines of AI-assisted Lean code proves machine-verification can outrun human understanding — and that LLMs will not fix the gap by producing better code, but only by forcing two structural rebuilds: deriving everything from formal specifications, and deliberately engineering diversity across implementations, models, and harnesses.

---

The essay opens with the certified AI-assisted proof of the Riemann hypothesis: correct, yet "no human (yet) understand[s] how it works." For Madhavapeddy this is the general condition of software finally made explicit — "There's always been an imbalance between our understanding of the theory of computation and our relentless drive to deploy software throughout society." The security symptoms (mysteriously correlated outages, supermarket fridges "needing James Bond to secure them") are decades of accumulated debt being called in, and he forecasts an "acceleration of security pain in 2027."

His answer is not better agents:

> "I don't think better harnesses and models *alone* will let software engineering catch up, as they just produce mountains of slop code."

Instead, two structural moves. First, formal specifications as the root artifact: translate legacy code by first spec-ing it, then derive all new software from specs — "rebuilding operating system stacks from the ground up and even reengineering instruction sets to match." Unikernels are his favoured substrate because fewer interfaces make the specification task tractable, and machine-checked fixes re-derived from a spec break the attacker/defender asymmetry: defenders stop waiting on maintainer triage. Second, deliberate diversity: since "any decent spec allows multiple valid implementations," agents should force genetic variation so a single bug can't take down vast numbers of hosts — while warning that "a handful of gated frontier models generating all our software takes us out of the frying pan into an AI controlled fire." Diversity must extend to models and harnesses themselves.

The residual human job: "Somebody still has to decide what our specs contain, as we have done for decades with RFCs" — plus new research into debugging heterogeneous fleets and assigning blame in them.

## Themes

#concept #specification #diversity #security

## Opinionated take

This is the strongest articulation yet in the wiki of specs as *load-bearing security infrastructure* rather than developer documentation — most spec-driven-development sources here treat specs as coordination tools for humans and agents; Madhavapeddy treats them as the only thing standing between society and a monoculture of machine-written code. The diversity argument is the genuinely novel move, and it cuts against the grain of current practice: the industry is consolidating on two or three frontier models, which by his logic is itself the vulnerability. Where the essay is weakest is the plausibility of "speedrunning" formal specs for the existing stack — an HN commenter puts it well: the spec *is* the hard part, and formalising decades of legacy behaviour may be strictly harder than rewriting it. The attached HN thread is a useful corrective: most commenters locate the failure in incentives (build-over-maintain economics, executives accepting risk) rather than human cognitive limits, which reframes his question "have we overestimated human ability?" as "have we underinvested in maintenance?" Both can be true, and the essay never really engages the incentive critique it invited.

## Relations

- Complicates [[The Agentic Product Standard v2.0]]: that standard's autonomy ladder and eval pyramid assume harness improvements drive quality; Madhavapeddy argues harnesses alone yield "mountains of slop code" and the binding constraint is the specification layer underneath.
- Strengthens [[Manifest-Driven Development]] from the opposite direction: CL Kao's "fake it until it works" gradual materialisation is the informal, pragmatic cousin of Madhavapeddy's demand that specs come first and code be *derived* — the same inversion of code and spec, at very different rigor levels.
- Nuances [[The Cathedral, the Bazaar, and the Winchester Mystery House]]: Breunig's attention-bottleneck analysis of the agentic era gets a darker corollary here — if machine-speed implementation outstrips human review, the fix isn't better review flow but removing humans from the verification loop entirely via machine-checked specs, with all the social-contract costs that implies for open source.
- Ties into [[Robustness Principle]] territory (and its HN critics within this very source): Madhavapeddy wants Postel's law inverted — hosts "strict in what we receive" — with diversity replacing liberality as the tolerance mechanism.

---
*Sources: [[raw/software-too-complex-for-humans]], [[summary/software-too-complex-for-humans]]*
*Last updated: 2026-11-26*
