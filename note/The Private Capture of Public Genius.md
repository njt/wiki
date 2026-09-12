# The Private Capture of Public Genius

Cameron Russell Armstrong makes the most coherent policy argument I've read for treating AI training data as a commons problem rather than a copyright one. Drawing a historical through-line from AT&T's 1956 consent decree to today's frontier labs, he argues that AI models are performing "the private capture of public genius at civilizational scale" and proposes a specific remedy: a corpus royalty — a fixed share of frontier lab gross revenue paid into a public fund and distributed equally.

---

## Key Quotes

> "Every dollar spent on research at Bell Labs did two things at once" — it was a recoverable cost *and* it expanded the rate base.

The essay's best historical insight. AT&T's regulated monopoly created a self-reinforcing cycle where R&D investment was both immediately recoverable and long-term value-building. The modern parallel is uncomfortably precise: frontier labs train on public data, capture the value in private weights, and face no equivalent obligation to reinvest in the commons they drew from.

> "Granting access is not the same thing as giving license."

The essay's thesis distilled to eight words. The frontier labs' fair use argument collapses the distinction between "this is public" and "you may take this." Armstrong is arguing that the legal system needs to recognize a third category between private property and unowned commons — something like *collectively owned but individually accessible*.

> "A person who read ten thousand books" returns sediment to the cultural delta slowly; a model "becomes a printing press that prints more printing presses."

The best rebuttal to the "just reading" defense I've encountered. The difference between human learning and AI training isn't metaphysical — it's a matter of speed, scale, and market effect. One person reading everything ever written becomes a slightly more interesting dinner guest. One model trained on everything ever written becomes a substitute for every writer it ingested.

> "You cannot pay people in proportion to their contribution because no administrable, specific share exists."

This is the analytical move that makes the corpus royalty proposal work. Armstrong concedes the measurement problem (Shapley values are computationally infeasible at frontier scale) but refuses to let that concession become an excuse for paying nothing. The impossibility of proportional attribution is not the impossibility of *any* attribution — it just means you need a different mechanism.

> Frontier labs seek "the privilege and power of a utility without accepting the public obligations that come along with it."

The closing line, and the essay's most quotable formulation. It frames the AI industry not as pirates stealing content but as utilities evading regulation — a more precise (and for the labs, more dangerous) framing.

## Key Themes

- **#pattern** — The AT&T consent decree as historical precedent for forcing technology companies to share the value of publicly-generated inputs
- **#concept** — *Corpus royalty*: a fixed revenue share paid into a public fund, sidestepping the impossible problem of per-contributor attribution
- **#concept** — *Attribution collapse*: the condition where individual contributions to a training corpus become simultaneously indispensable in aggregate and impossible to value individually
- **#concept** — The excludability/rivalry taxonomy applied to internet data: frontier labs treat public data as a public good for extraction purposes but as a private good for monetization
- **#pattern** — The Superfund analogy: you don't need to trace specific barrels of waste to specific polluters to hold the industry accountable for aggregate harm
- **#comparison** — Ostrom's commons governance conditions as a diagnostic for why the internet is failing as a commons: hazy boundaries, no defined community, no graduated sanctions

## Critical Analysis

Armstrong has written the policy essay the AI copyright debate has been waiting for. Most arguments in this space fall into two camps: "training is theft, pay everyone" (impossible to administer) or "training is fair use, pay no one" (morally and economically dubious). The corpus royalty splits the difference in a way that's both principled and practical.

**What works:**

The AT&T consent decree is a genuinely illuminating historical analogy. It's not just decorative — Armstrong uses it to establish that (a) forcing technology companies to share publicly-generated value has precedent, (b) doing so can *accelerate* an industry rather than kill it, and (c) the mechanism can be blunt rather than precise and still work. Gordon Moore himself crediting the decree with launching Silicon Valley is the kind of testimony that makes policy proposals feel inevitable rather than radical.

The Superfund analogy is similarly shrewd. Environmental law already solved a structurally identical problem: how do you hold an industry accountable for aggregate harm when tracing individual contributions is impossible? The answer was strict liability + an industry-funded cleanup fund. Armstrong is essentially proposing Superfund for the information commons.

The Ostrom section is the essay's hidden depth. By mapping the internet against Ostrom's eight conditions for sustainable commons governance and finding it fails nearly all of them, Armstrong frames the corpus royalty not as a punishment but as a *governance mechanism* — the thing you add when self-regulation has demonstrably failed.

**What's underdeveloped:**

The essay is explicitly "Essay I of Upstream of Everything," and it shows — the corpus royalty is gestured at more than specified. What percentage of gross revenue? Which labs count as "frontier"? How do you prevent labs from structuring around the threshold? Does the fund distribute to citizens of a specific country or globally? These are tractable questions, but the essay doesn't answer them.

The Alaska Permanent Fund analogy cuts both ways. The APF works because oil is geographically bounded — you can point to a specific patch of Alaskan ground and say "this resource belongs to these people." Training data has no such boundary. If the corpus royalty is truly universal, it becomes a global UBI funded by AI revenue, which is a much bigger political lift than the essay acknowledges.

The essay also dodges the hardest question: why should the public get *equal* shares rather than shares proportional to some measure of contribution (GDP, population, internet usage, content production)? Armstrong treats equal distribution as the natural default once proportional attribution is impossible, but that's a political choice, not a logical necessity. China and the EU will have opinions about a royalty fund that disproportionately benefits Americans.

**The unspoken argument:**

The essay doesn't say this explicitly, but its structure implies it: the corpus royalty isn't really about compensating individuals for their training data contributions. It's about preventing the frontier labs from capturing *all* the surplus. The egalitarian distribution is a feature, not a bug — it dissolves the attribution problem by refusing to solve it. The point is to take money from the labs and give it to the public; the "corpus" framing is the justification that makes it legally and politically legible.

Whether that's honest or cynical depends on your priors. I lean honest — Armstrong seems genuinely committed to the commons framework. But the essay works equally well read either way, which is either its strength or its limitation.

## Connections

- [[Who Owns the Code Claude Wrote]] — The IP questions Armstrong raises about training data are the input-side version of the output-side questions Evren maps for AI-generated code. Both converge on the same problem: our legal frameworks assume human-scale creation and human-scale attribution, and AI breaks both assumptions simultaneously.
- [[Eye of the Master]] — Pasquinelli's thesis that AI is fundamentally a technology for extracting workers' knowledge and crystallizing it into capital-owned machines is the Marxist version of Armstrong's argument. Armstrong arrives at the same destination through property law and commons theory rather than labour theory; both are saying the same thing in different vocabularies.
- [[AI Will Not Make You Rich]] — Neumann argues AI's economic value will flow to customers and consumers, not builders. Armstrong is essentially proposing a mechanism to make that happen by force rather than by market dynamics. If Neumann is right about the containerization thesis, the corpus royalty is unnecessary; if he's wrong, it's essential.
- [[The Dead Economy Theory]] — McGrann's framing of an economy that hums along without human participation is the dystopian endpoint Armstrong is trying to prevent. The corpus royalty is, in McGrann's terms, a way to keep "present people" as the unit of account even when they're no longer the unit of production.
- [[Dopamine Fracking]] — German S.'s extraction metaphor maps cleanly onto Armstrong's argument: the frontier labs are "fracking" the cultural commons, extracting value at industrial scale with no obligation to replenish what they've taken. The corpus royalty is a severance tax on information extraction.
- [[The Future of Everything is Lies I Guess]] — Aphyr's comprehensive catalog of LLM harms is the empirical backing Armstrong's policy argument needs but doesn't provide. Where Armstrong is making a property-rights argument, Kingsbury is documenting the actual damage; together they make a stronger case than either does alone.
- [[AI Value Chain]] — Armstrong is essentially arguing that the value chain is broken at its source — the training data layer — and proposing to fix it by inserting a new extraction point upstream of the labs themselves.

---

*Sources: [[raw/private-capture-of-public-genius]]*
*Last updated: 2026-07-08*
