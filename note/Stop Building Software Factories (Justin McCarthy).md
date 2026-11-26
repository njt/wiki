# Stop Building Software Factories (Justin McCarthy)

Justin McCarthy's ai.engineer 2026 talk delivers a provocation with a twist: the founder of a software-factory company tells the room to stop building software factories — because factories only work where demand is already proven. His distinction between a **brownfield factory** (continuous agent loops keeping existing promises on products customers already pay for) and **greenfield discovery** (where you don't know what you want yet and shouldn't industrialise) is the talk's real contribution, before it escalates into a call for a "demand factory" to soak up the software glut.

---

## Key quotes

> "Please stop building software factories... most of the time we don't know what we want... if we're trying to industrialize something where we don't know what we want, we're certainly not going to get the right thing or we're going to take too long or we're going to spend too much."

The title is clickbait with a precise edge. He isn't anti-factory; he's against applying factory rigor to discovery work, where the feedback signal doesn't exist yet. The discernment question — are we conquering new territory or doing something with precedent? — is the whole decision.

> "If you cut that feedback leg, you're just going to amplify noise."

His amplifier-block-diagram metaphor is the cleanest one-line account of why verification loops matter: the loop isn't automation theater, it's the thing that turns agent output into stable input. Without fidelity fed back, you get confident noise at scale — which matches what the wiki's loop-engineering sources keep finding empirically.

> "The skills are getting shorter, of course. The harnesses are getting more general, of course... you can expect a smoother, laminar flow, higher performance soon, without much of your creative contribution in between."

The Raptor-1→Raptor-2 argument is the talk's most interesting claim: factories **self-simplify** because the model is the improving component. Build the janky v1 that barely works, and the next model version deletes tubes for you. That inverts the usual maintenance instinct — over-engineering the factory now is a bet against model progress.

> "They weren't asking for software. They were asking for more gross margin, more customers, more brand differentiation."

This is the pivot to the second half: the brownfield factory produces a glut of working software that maps onto nothing the customer actually wanted. Hence the "demand factory" — agents that ask better market questions, synthesise answers overnight, re-rank, and schedule price-elasticity experiments. It's deliberately half-formed ("I don't know what a demand factory is") and that honesty is part of the point: this is the next discovery frontier.

> "Figure out what your energy drink is and start selling it now."

The brand-envelope framing (Spindrift: caffeine yes, motor oil no) is a genuinely useful product heuristic dressed in comedy — the maximum credible promise you can make adjacent to what you already are.

## Themes

#concept #pattern #person #business

## Opinionated take

This is the best-structured *boundary* argument in the wiki's factory corpus. Where most software-factory sources answer "how," McCarthy answers "when" — and his when is unflattering to the genre: factories are for the boring, revenue-proven part of the estate, and the urge to factory-ise your new idea is exactly backwards because discovery needs looser loops and human taste. The "same but faster, cheaper, safer — but not better" instruction is a sharp articulation of where agent value ends and human judgment begins.

The talk has two weaker joints. First, the demand factory is hand-waved almost entirely — "rate match demand systems to production systems" is a slogan, not a design, and the slide from customer questions to "acquire your competitor" reads as keynote bluster. Second, "it takes like a week" undersells the brownfield-structure problem badly; the practical sources in this wiki (Osmani, Pupius) document months of characterization tests and zone maps before agent loops are trustworthy in legacy code. McCarthy is selling, and the audience is exactly the buyers for it.

Still, the simplification-over-time claim deserves to be taken seriously and tested: it implies factory harnesses should be built deliberately janky and disposable, which is a genuinely contrarian design stance.

## Related

- [[Two Jobs — Product Engineering and Factory Engineering (Lloyd)]] — Zach Lloyd's "build the product / build the factory" split maps directly onto McCarthy's greenfield/brownfield split; McCarthy adds the demand side that Lloyd's frame leaves implicit, and both agree the factory is the durable artifact.
- [[What Matters After the Software Factory Works]] — this source strengthens that page's argument that a working factory is the floor, not the finish line: McCarthy's answer (build a demand factory, escape the software glut) is more aggressive than most post-factory agendas.
- [[Loop Engineering]] — McCarthy's amplifier-feedback-leg framing is a physics-flavoured restatement of why loops need verified signal fed back; it nuances the loop-engineering corpus by stressing that the loop's justification is *demand-proven*, not novelty.
- [[Brownfield Agentic Engineering (Osmani)]] — complicates it directly: Osmani documents weeks-to-months of structure (zones, characterization tests) before agents are safe in legacy code, where McCarthy claims a brownfield factory takes "like a week."

---
*Sources: [[raw/justin-mccarthy]], [[summary/justin-mccarthy]]*
*Last updated: 2026-11-26*
