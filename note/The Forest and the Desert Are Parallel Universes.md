# The Forest and the Desert Are Parallel Universes

Kent Beck's Craft 2025 keynote resolves the mystery that has haunted his career — why do spectacularly successful software teams get fired? — with a framework: "forest" and "desert" are not points on a spectrum but parallel universes, each a self-consistent belief system (Rousseau and Follett's "power with" versus Hobbes and Taylor's coercion and the one-best-way) in which the same words — estimates, metrics, dependencies, compliance, accountability, status — mean opposite things. The desert optimizes for managing the work; the forest optimizes for doing it. The talk is a diagnostic rather than a conversion pitch: Beck appreciates the desert on its own terms, but he is not interested in thriving in it.

---

## Key quotes

> "It's not that the teams doing software badly are doing software badly, it's that they're doing something else very well. That something else just doesn't happen to be software development."

The thesis, from the origin story: at XP Universe around 2000–2001, Beck counted five teams in the audience that were spectacularly successful by their organizations' standards — "more code in less time, with no bugs, happy customers" — and all five were fired. This is the sentence that reframes twenty-five years of failed methodology advocacy: the advocacy wasn't wrong, it was arguing quality to organizations optimizing for something else entirely.

> "The problem's not the problem. The problem is the context."

The one-line summary of why "we know how to do this better" arguments never land. Every technique debate — no estimates, metrics, dependencies — is actually a collision between two contexts, and each side hears the opposite of what was said. This is the talk's most transferable idea: before advocating anything, read which universe you're standing in.

> "You never plan for more than half of what you think you can accomplish. Never."

A forest rule the desert hears as "malfeasance, that sounds like slackers." The freed time goes to community: helping teammates, learning something and bringing it back, building structures "orthogonal to the formal org chart." The rule is only crazy if you assume the plan's second half would have been achieved anyway — which, Beck says, it won't be.

> "If you could get done with 10 people what you're currently getting done with 1000, then your status at the country club goes way, way down."

The most cynical and most useful line in the Q&A. Beck predicts executive resistance to AI-driven team shrinkage regardless of what the tools can do — status is denominated in reports, so efficiency is a personal loss even when it is an organizational win. This is the mechanism behind the incentive wall he described in [[Martin Fowler and Kent Beck on Reinventing Software]], now with a name.

> "Is the forest scalable? I think it's the only thing that is scalable."

Beck inverts the usual question. As organizations grow, management becomes a bigger and bigger job, so "the organization is more and more optimized for management, not for the job itself." Scale is precisely what manufactures desert; asking whether the forest survives scaling gets the causality backwards.

> "I'm not interested in thriving in a desert. I want to find ways of moving to the forest."

His dismissal of "Stockholm syndrome bloggers" who publish guides to thriving under stack ranking. Naming the genre is a service: adaptation literature that optimizes individual survival inside a perverse system is, from where Beck stands, collaboration with it.

> "If you design, you'll have time to design."

The resource-creation loop in one sentence, from his Bluesky aphorism. The desert sees resources as a fixed supply to ration; the forest sees them as something "to be created and shared" — spend time on the structure of the system, get time back, spend it on structure again. It is the compounding argument for design stated without a single architecture word.

> "The one where you get the fruits without the roots, it just doesn't happen."

His answer to the top-voted question — management wants forest results while acting in desert ways. Drawing on the "We Know How to Develop Software Well" presentation he did with Beth: everyone wants the fruits (no bugs, happy customers, sooner, cheaper), but the fruits come with roots (the practices), and there is no third option. You can be a good desert organization or fund a forest one; fruits-without-roots is not on the menu.

> "how many bits of information is here? Zero. And yet from a desert perspective, the fact of answering has value. I know when I can punish the people who do not meet this deadline."

The 87-government-agency story, where Beck refused for a year to give a date for an open-ended project. The same utterance is zero information in one universe and a punishment schedule in the other — the crispest demonstration in the talk that "the same words don't mean the same thing at all."

## Key themes

- #concept — **Two attractors, not a spectrum.** "It is not just a matter of degree." The forest and desert are stable belief systems, and the desert has more gravitational pull: "the natural tendency is always going to be to move from the forest into the desert." Oases are satisfying but temporary without constant care.
- #concept — **Same words, opposite meanings.** Estimates, metrics, dependencies, compliance, accountability, and status are "lightning rods." In the desert, metrics measure performance and dependencies diffuse blame (enabling "scheduled chicken"); in the forest, metrics are for self-awareness and cross-team concerns get resolved "at the boundary between the teams."
- #pattern — **Half-full planning.** Deliberately underfill the plan and spend the remainder on community and learning — capacity the desert reads as slacking is the forest's investment mechanism.
- #pattern — **Metrics for self-awareness, never goals.** Beck's TDD-book experiment (timestamping every test run, discovering he tested "kind of once a minute" when once a day was considered fast) is the model use. Rules: metrics "are always gamed if they've turned into goals," and individual metrics are never reported two levels up — only team-level numbers rise, to prevent distortion.
- #pattern — **Compliance as a resource.** Square Life compressed a "year-long hundred million euro process" into a weekend, "including auditing, regulatory compliance, the whole thing... and there wasn't coding on the weekend." In the desert, compliance's best use is "a good excuse not to improve."
- #person — **Kent Beck** as the XP founder who spent decades watching his ideas win arguments and lose organizations. **Mary Parker Follett** ("power with" vs. "power over") and **Frederick Winslow Taylor** (the fudged-data "one best way") as the two philosophical parents; "Taylor won — it's still only a century."

## Critical analysis

**The diagnosis is the product, and the talk is honest about that.** After twenty-five years of failed advocacy, Beck has stopped selling and started explaining. That is the right call — the word-by-word translation table (what "accountability" or "status" means in each universe) is the talk's most durable artifact, a diagnostic you can actually use in a meeting. But the move from diagnosis to action is undersupplied: no stay-versus-leave recommendation ("I'm not going to give you a blanket recommendation"), no transformation roadmap, and — glaringly — no advice for individual contributors. Every Q&A answer is managerial. The one IC question he names (a developer told not to write tests "because we have little people to do that for us") is the one he never answers.

**"Not a matter of degree" is asserted, not argued.** This is the load-bearing claim and it rests on vibes. Hybrids plainly exist: incident response is legitimately desert-shaped (predictability under crisis), hard regulatory launches legitimately reward plan conformance. The binary is a superb lens and a poor ontology, and the evidence base — five teams circa 2000, one Swiss insurer, one daughter's manager — is anecdote-strength for a claim about all organizations. A friendlier reading: the "two attractors" framing concedes mixture exists (attractors, not kingdoms) while insisting mixture is unstable. That is defensible, but the talk doesn't make that argument; it just repeats the assertion.

**The unnamed optimization.** The talk's central promise — the desert is "doing something else very well" — is never cashed out concretely. Blame diffusion, control, and executive status are the implicit candidates, and the country-club quote half-admits the desert is a status machine. If Beck named it plainly — the desert optimizes for status preservation and blame avoidance — the framework would be sharper and falsifiable. Leaving it metaphorical keeps the talk comfortable for exactly the desert executives he says he now talks to with his "cake strategy."

**The AI hole is the most interesting omission.** Beck is "spending a lot of my time on augmented coding" and says "the answers are all changing," but never connects AI to his own framework — even though his country-club line does the connecting for him. AI is the strongest test the desert has faced: keystroke metrics are a desert instrument par excellence, while agent-afforded throwaway exploration ("we spent a week on it, threw it away") is forest fuel. Which way it tips depends on which universe adopts it — and the desert will adopt it first, because the desert buys tools, not beliefs. The forest/desert lens predicts that AI lands in most organizations as measurement, not exploration.

**What is genuinely new: appreciation without endorsement.** Most craft writing moralizes — good engineers versus Dilbert bosses. Beck refuses: desert people succeed "in spite of all the obstacles they put in their own way. It's amazing. Good on you." That stance is what makes the diagnostic usable; you cannot read your own organization accurately while despising it. It also separates this talk from the craftsmanship-sermon genre it superficially resembles — and from the trope that the desert is merely a failed attempt at the forest. The desert is a success at its own game. That is the disturbing part.

## Related pages

- [[Martin Fowler and Kent Beck on Reinventing Software]] — the same speaker, months earlier, observing that organizations "will punish you" for faster/cheaper/better; this keynote is the full theory behind that one-liner. The punishment is not irrational — it is desert-coherent — and the Fowler/Beck conversation's unanswered question about incentive walls gets its mechanism here.
- [[Lean Software Production]] — complicates it. Matt Wynne argues XP practices are now survival equipment that any competent org should adopt; Beck explains why those same practices are heard as malfeasance in desert organizations. The adoption problem is a context problem, not an argument-quality problem — "the problem's not the problem."
- [[Building World-Class Engineering Teams in the Age of AI]] — complicates it. Rajeev and Dohmke report "89% more PRs" and predict flatter organizations; Beck's country-club status economics predicts exactly why the headcount savings never get banked, and why "how many reports do you have?" remains the executive status currency no efficiency dashboard can devalue.
- [[In Praise of Normal Engineers]] — strengthens it. Charity Majors' talk from the same Craft 2025 stage (teams own software, orgs are defined by their floor) is forest rhetoric in practice; Beck supplies the meta-framework for why it lands with some audiences and bounces off others.

---
*Sources: [[raw/the-forest-the-desert-are-parallel-universes-kent-beck-craft-2025]], [[summary/the-forest-the-desert-are-parallel-universes-kent-beck-craft-2025]]*
*Last updated: 2026-09-13*
