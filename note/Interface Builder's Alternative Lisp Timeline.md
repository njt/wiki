# Interface Builder's Alternative Lisp Timeline

Paul Hammant reconstructs a counterfactual lineage of visual UI tooling: ExperTelligence's 1986 Lisp-based Interface Builder (shown to Steve Jobs, ancestor of NeXT's IB) and its 1988 successor Action!, a Texas Instruments product whose UI was a live, runtime-modifiable Lisp program saved as human-readable, source-control-friendly Lisp. The essay is equal parts oral history, markup-format taxonomy, and "what if the web had been Lisp" speculation — punctuated by Denny Bollay's confession that his far-superior hypertext system lost to HTML.

---

## The story

Denison "Denny" Bollay's ExperTelligence team built the first Interface Builder in 1986, saved as binary Mac resource forks, and demoed it to Steve Jobs — Jobs went from sceptical (vs. unreleased HyperCard) to converted in thirty minutes. A plan to license it to NeXT collapsed when a team member left and "started over on the same product" there. In 1988, Bollay built **Action!** with Texas Instruments: an interface "fully modifiable **AS** the program was running," in incrementally compiled Lisp, demoed to Bill Gates. Microsoft wanted a version in Basic; Bollay told them that was impossible without objects — they built Visual Basic instead.

Action!'s key property is that its markup *was* its program: Lisp, interpreted identically at design time and runtime, round-trip editable in Vi/Emacs, and — Hammant argues — source-control compatible from day one. Nothing since, he claims, has matched its sophistication 25 years later.

> "To me, Action! is one the 'holy grail' items from the history of user interfaces, if not Information Technology more generally."

This is the essay's thesis in one line: a live, unified, inspectable representation of both UI structure and behavior — the thing the DOM/HTML/JS triad still approximates clumsily.

## The counterfactual

The most delicious digression: Berners-Lee built the first browser *in* NeXT's Interface Builder. And the CIA, after seeing Action!, funded Bollay's "Dynamic Documents" — object-oriented hypertext with contextual, bidirectional, out-of-band links between arbitrary documents, stored in a Lisp object database. Shown an early HTML spec, Bollay's team (with Xerox PARC, of NoteCards fame) concluded it was "dumb from almost every angle": re-authoring required, links non-contextual and unidirectional, static, style mixed with semantics, "even HTTP was stupid."

> "Obviously we misjudged…"

This is the best-earned quote in the piece. It's not nostalgia — it's an object lesson that technical superiority is nearly irrelevant to platform selection. HTML won on simplicity, openness, and network effects, and the elegant alternative is a garage-legend. Hammant runs the same question through ChatGPT in 2023 ("would a Lisp-nib NeXT have given us Lisp instead of HTML?") and gets a plausible shrug: probably things converge to an SGML-ish markup anyway.

## Why it matters now

The Action!/DOM comparison reads oddly prescient from 2026: Action! had a live mutable model of UI objects, instant design↔run flipping, and bindings to high-level languages — but no URLs, no pages, no XPath, no client/server. Hammant's "what if?" is really a question about which abstractions the web *omitted*, and whether live-editable, round-trippable structure is a permanently lost alternative or one that keeps resurfacing (in live-reload tooling, in htmx-style hypermedia revivals, in LLMs generating UI as data).

## Analysis

Hammant is honest about his agenda (he calls it an anti-AngularJS "pseudo-declarative UI" hobby) and the piece has the texture of a blog post chasing a rumor down — charming, a little credulous (one side of the story, mostly Bollay's email), and lightly sourced. But the historical claims are the checkable kind: demos to Jobs and Gates, the .nib/.xib lineage, WorldWideWeb's NeXT provenance. The real contribution is framing markup formats as *choices with survivors*: binary resource forks → nib → xib → and the road not taken, Lisp. Every generation rediscovers that human-readable, diff-able, tool-editable representations age better than binary ones; Action! simply got there in 1988.

The lost lesson for the agentic era: Bollay's Dynamic Documents didn't fail because contextual out-of-band links were a bad idea. It failed because its model required buy-in from a platform owner, while HTML required buy-in from nobody. Technical quality is not the selection pressure in networked systems — friction is. That's a caution against reading any modern "holy grail" protocol as inevitable.

## Themes

#concept #tool #person

## Related

- [[Lost Golden Age of Languages]] — Hammant's Lisp-elegy is a concrete instance of that page's thesis: Action!'s Lisp DSL is exactly the kind of expressive 1980s-language capability that mainstream tooling abandoned, and this source gives it a specific artifact and date.
- [[hypereffect]] — The modern echo: a tiny fixed component vocabulary with structured expressions as a shared interface layer for agent-generated UIs, replaying Action!'s bet that a minimal inspectable markup beats sprawling binary formats.
- [[Components of a Hypermedia System]] — The web that won: Fielding's uniform-interface hypermedia model is the descendant of the HTML spec Bollay's team judged "dumb," which makes this source the perfect complicating footnote — the winner was, by their criteria, the worse design.

---
*Sources: [[raw/interface-builders-alternative-lisp-timeline]], [[summary/interface-builders-alternative-lisp-timeline]]*
*Last updated: 2026-11-11*
