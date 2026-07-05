# The Future of Everything is Lies I Guess

Kyle Kingsbury (Aphyr) -- the Jepsen distributed-systems-testing guy -- wrote a 10-part essay series in April 2026 cataloguing the ways LLMs will deform society. Not a hype piece, not a doomer manifesto: a methodical, technically grounded survey of second- and third-order effects, from chaotic dynamics and information ecology collapse to deskilling, psychological hazards, and the consolidation of power in ML megacorps. It is the most comprehensive skeptical treatment of LLMs published to date, and it lands hardest when Kingsbury brings his systems-correctness expertise to bear on the verification problem.

---

## The Arc

**1. Introduction.** LLMs are "sophisticated generators of text" that confabulate constantly, exhibit a "jagged technology frontier" of competence, and whose "reasoning" explanations are "predominantly inaccurate." The fundamental question of whether scaling produces real gains remains unanswered.

**2. Dynamics.** LLMs are chaotic systems: small input changes produce unpredictable output changes. This creates illegible attack surfaces, strange attractors (models getting stuck in loops), and a verification problem -- output looks plausible, so errors hide. The killer insight: LLM-generated code provides short-term velocity but compounds complexity and fragility, citing real outages at GitHub (sub-90% uptime) and AWS.

**3. Culture.** We lack cultural scripts for what LLMs actually are. Sci-fi mythologies are wrong; Peter Watts' Blindsight is closest. LLMs will create new media forms (interactive model-as-book), reshape sexuality, and produce recognizable "slop" aesthetics that will be appropriated and ironized. "General-purpose ML companies are intrinsically tasked with encoding, formalizing, and adjudicating essentially all cultural norms."

**4. Information Ecology.** ML scrapers degrade the open web. LLM-generated content destroys trust signals. Spam scales effortlessly. State propaganda industrializes. Search results fill with slop. Visual evidence loses reliability. Kingsbury invokes Hannah Arendt: audiences learn to "believe everything and nothing."

**5. Annoyances.** Customer service becomes endlessly polite machines giving unreliable answers. ML creeps into "fuzzy" decisions (insurance, parking, pricing). Corporations absorb LLM unpredictability at scale; individuals eat the variance. Accountability diffuses until "no one is directly responsible."

**6. Psychological Hazards.** Models optimized for engagement become Skinner boxes with intermittent reinforcement. Lonely people get always-available companions that never challenge them. Children get "cogitohazard teddy bears." Jane Jacobs' "ubiquitous, casual relationships" get displaced by algorithmic companionship.

**7. Safety.** Alignment is optional and expensive; unaligned models are inevitable. LLMs enable exploit discovery, fraud amplification, harassment at scale, and autonomous weapons. The "lethal trifecta" is really a "unifecta" -- LLMs shouldn't wield dangerous power at all.

**8. Work.** Software development may become "witchcraft" -- elaborate incantations to fickle daemons. "Skills files become spellbooks." Bainbridge's 1983 "Ironies of Automation" applies directly: deskilling, monitoring fatigue, takeover hazards. Capital consolidates; UBI from AI megacorps is "hopelessly naive."

**9. New Jobs.** Six new roles: Incanters (prompt specialists), Process Engineers (QC workflows), Statistical Engineers (measuring model variability), Model Trainers (the "largest harvesting of human expertise ever attempted"), Meat Shields (Elish's "moral crumple zone"), and Haruspices (post-mortem model analysts).

**10. Where Do We Go From Here.** The automobile analogy: we celebrate speed while ignoring structural deformation. Preserve "metis" (craft knowledge). Write your own words. Unionize against Copilot mandates. Slow down.

## Key Quotes

> "They work by predicting statistically likely completions of an input string, much like a phone autocomplete."

> "Reasoning models write fanfic about themselves."

> "Not one produced error-free summaries." (on clinical AI scribes)

> "LLMs perform identity, empathy, and accountability -- at great length! -- without meaning anything. There is simply no there there."

> "Skills files become spellbooks."

> "General-purpose ML companies are intrinsically tasked with encoding, formalizing, and adjudicating essentially all cultural norms, and must do so at unprecedented scale."

> "The ML industry is creating the conditions under which anyone with sufficient funds can train an unaligned model."

> "I have never used an LLM for my writing, software, or personal life, because I care about my ability to write well, reason deeply, and stay grounded in the world."

> "An uncanny, flattened simulacra is part of the fun." (on drone kink aesthetics)

> "What's called 'AI slop' today will become the Frutiger Aero of 2045."

## Key Themes

#technology-criticism #llm-skepticism #deskilling #information-ecology #automation #political-economy #safety

## Critical Analysis

**Where Kingsbury is strongest:** The dynamics chapter is where his Jepsen pedigree shows. His treatment of LLMs as chaotic systems -- attractors, illegible hazards, the verification problem -- is the most technically rigorous part of the series. The latent disaster argument (short-term velocity masking compounding fragility) maps directly onto distributed systems failure modes he's spent a decade documenting. When he says LLM-generated code will produce "latent disaster," this is the Jepsen guy telling you your system has a linearizability bug that hasn't triggered yet. Listen.

The "ironies of automation" thread through Work is also excellent. Connecting Bainbridge's 1983 framework to LLM coding assistants is not original (others have noted it), but Kingsbury executes it with concrete specificity -- software engineers feeling less able to write code, doctors worse at spotting adenomas -- that makes the abstraction real.

**Where he's writing outside his domain:** The Culture chapter is ambitious and sometimes brilliant (the slop-as-aesthetic section is genuinely fun, the pornography analysis is refreshingly frank), but the media-theory sections are undergraduate-level McLuhan. The "models as new media" speculation about cooking-instruction LLMs and personality terraria is interesting but underdeveloped compared to the systems analysis.

The Information Ecology chapter leans heavily on "everything is getting worse" without engaging seriously with countervailing evidence. Trust in media was declining long before LLMs. The Arendt invocation is apt but also easy -- everyone quotes Arendt when discussing epistemological collapse.

**What's missing:** Almost no engagement with the people actually building and shipping LLM products responsibly. The wiki's own pages on [[Compound Engineering]], [[Guardrails and Feedback Loops]], and [[Harness Engineering]] document real engineering practices that mitigate many of the hazards Kingsbury describes. He acknowledges "some peers report their LLM projects maintained controlled complexity through careful gardening" and then moves on. A stronger version of this essay would grapple with why some teams succeed and others don't, rather than treating all LLM use as a single undifferentiated mass heading toward disaster.

No mention of evals, benchmarks, or structured quality frameworks. The wiki's [[Demystifying Evals for AI Agents]] and [[Feedback Loop is All You Need]] represent an entire engineering discipline Kingsbury ignores. His "witchcraft" framing is vivid but unfair to teams doing serious [[Harness Engineering]].

The labor analysis is also thin relative to existing work. [[Eye of the Master]] provides a deeper historical framework for AI-as-labor-extraction; [[Why We Fear AI]] makes a sharper version of the capitalism-not-technology argument. Kingsbury reaches similar conclusions but with less theoretical depth.

**How it connects to the wiki:** This series is the single best articulation of the skeptical position against which the wiki's practitioner-oriented content should be read. The [[Cognitive Debt]] concept maps directly to Kingsbury's "latent disaster" argument. His verification problem is the problem [[Compound Engineering]] and [[Guardrails and Feedback Loops]] exist to solve. His "programming as witchcraft" is what [[Feedback Loop is All You Need]] argues against -- linters beat incantations. The [[acceleration-flow]] page's slot-machine metaphor anticipates his "Pandora's Skinner Box" almost exactly. His new-jobs taxonomy (especially Haruspices and Process Engineers) maps to roles the wiki already documents in [[Demystifying Evals for AI Agents]] and [[Harness Engineering]].

The series is essential reading for anyone in the wiki's audience precisely because it forces you to articulate *why* your engineering practices are sufficient against the failure modes he describes. If you can't answer Kingsbury, you probably shouldn't be shipping.

## Related Pages

- [[Cognitive Debt]] -- the same "velocity exceeds comprehension" thesis, from the practitioner side
- [[Write Only Code]] -- Slop Radius as the engineering response to Kingsbury's verification problem
- [[acceleration-flow]] -- the slot-machine/dopamine loop he describes in Psychological Hazards
- [[Eye of the Master]] -- deeper historical framework for AI-as-labor-extraction
- [[Why We Fear AI]] -- sharper version of "this is capitalism anxiety, not technology anxiety"
- [[Compound Engineering]] -- the engineering answer to his latent-disaster argument
- [[Guardrails and Feedback Loops]] -- linters beat incantations
- [[Harness Engineering]] -- the systematic framework for the verification problem
- [[Feedback Loop is All You Need]] -- "your CLAUDE.md is a suggestion; your linter isn't"
- [[Demystifying Evals for AI Agents]] -- the eval discipline his New Jobs chapter reinvents
- [[Security and Sandboxing]] -- architectural constraint vs. his "unifecta" argument
- [[Distributed Systems]] -- Kingsbury's home turf; his dynamics chapter IS distributed systems failure analysis
- [[HackAPrompt Dataset]] -- the illegible-hazards attack surface he describes
- [[Cybersecurity Is Proof of Work Now]] -- security as compute economics, extending his Safety chapter
- [[The Next Two Years of Software Engineering]] -- data on the labor displacement he fears
- [[Slowing the Fuck Down]] -- the practitioner version of his "slow down" conclusion

---
*Sources: [[summary/the-future-of-everything-is-lies-i-guess]]*
*Last updated: 2026-05-14*
