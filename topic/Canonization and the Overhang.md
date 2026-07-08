# Canonization and the Overhang

Kellan Elliott-McCrea maps David Bessis's diagnosis of mathematics onto software engineering, coining two terms that name something every experienced engineer already feels: **canonization** (turning one-off code into reusable, coherent library-grade work) and **the overhang** (latent value from connecting existing work, a dividend of canonization). The core warning: if we reward only production and not canonization, we eat our seed corn — the overhang depletes, and the search space LLMs mine grows poor. Published June 2026, responding to Bessis's "The fall of the theorem economy."

---

## Key Quotes

> "Canonization: taking a local, one-off formalization and turning it into library mathematics: general, reusable, coherent, efficient, and compatible."

Kontorovich's definition, borrowed by Elliott-McCrea. This is the work that separates "code that passes tests" from "code the future can build on." It's what the old aesthetic standards of software engineering were implicitly doing — canonization makes it explicit.

> "We're in a golden age of disposable single-use software that reads as 'bad' by older aesthetic standards."

This is the sharpest observation in the piece. Vibe-coded software works but doesn't accrete. The aesthetic judgment isn't snobbery — it's a proxy for whether code is canonized. When vibe-coded solutions feel *more* reliable than established ones, that's a canonization failure, not a vibe-coding victory.

> "If we value only the proving, and not the canonizing nor the training of new practitioners, the Cognitive Surplus becomes Cognitive Seed Corn."

The pivot from surplus to seed corn is the essay's emotional core. Review grows fatiguing, cleanup earns no status, glue work is invisible — and the overhang, the very thing that makes LLM-powered search of the accumulated corpus valuable, grows thin.

> "Generation easy. Canonization hard."

The thesis compressed into four words. This belongs next to "code is disposable, specs are the durable artifact" and "the product is still working software, but the work is engineering the system that produces it."

---

## Key Themes

- **#pattern** — Canonization as the explicit name for what craft standards were doing implicitly. Separates accretive work from disposable work.
- **#concept** — The Overhang as latent value from past creativity, a dividend of canonization. Software's overhang: open source, Stack Overflow, blogs, decades of public repos.
- **#pattern** — The photo negative of technical debt: features that resist change and drain morale indicate absent canonization, not just accumulated shortcuts.
- **#concept** — Cognitive Seed Corn: the social production of knowledge as a resource that must be replenished, not merely extracted. Training practitioners and canonizing work are the replenishment mechanisms.

---

## Critical Analysis

This is one of the best essays of 2026 on what AI coding actually changes — and what it doesn't. Elliott-McCrea does the rare thing of naming something you already felt but hadn't articulated.

The canonization frame solves a real problem: the "vibe coding vs. craft" debate has been stuck because both sides are right. Vibe coding produces working software fast; craft produces software that compounds. Canonization tells you *when* to switch — write disposable code to learn, canonize when you need to build on it. This is more useful than either pole.

The overhang concept is genuinely novel and worth stealing. It names why LLMs are good at certain kinds of discovery (pattern-matching across an entire corpus) and why maintaining that corpus matters. The overhang isn't just a static archive — it grows and shrinks with the quality of canonization feeding it. This makes "clean up that code" not a matter of taste but of resource management.

The weak point: Elliott-McCrea doesn't grapple with *who pays* for canonization. Glue work has been undervalued since before AI. The essay names the problem — "reviewing grows more fatiguing and unrewarded, cleanup earns no status" — but offers no mechanism for changing the incentives. The unspoken question: if the market won't pay for canonization, who will? The answer might be "the people who depend on the overhang," which means everyone, which means nobody in particular.

The piece gains force from its brevity. It doesn't over-build the analogy or try to turn canonization into a framework. It names two things, draws the connection, and leaves the implications for you to work out. That restraint is itself a form of canonization — leaving space for others to build on the idea.

---

## Related Pages

- [[Specifications as the Product]] — Code is disposable; specs are the durable artifact. The economics have inverted. Directly parallel to "generation easy, canonization hard."
- [[Vibe Coding and the Maker Movement]] — Evaluative anesthesia: the dopamine of making eclipses the ability to judge. The psychological mechanism behind disposable single-use software.
- [[Vibe Coding as a Team Sport]] — Jon Udell's constructive answer to vibe coding chaos. Process as the path from disposable to canonized.
- [[The Cost YAGNI Was Never About]] — Kent Beck reframes YAGNI as options pricing. Cheap AI generation amplifies the trap of building things you shouldn't, not the escape.
- [[Lean Software Production]] — Matt Wynne's framework for the post-craft era. The product is still working software, but the work is engineering the system that produces it.
- [[Software Engineering Craft]] — Fundamentals that don't change. Canonization is one of them.
- [[Agent Coding Workflow]] — Maturity spectrum from vibes to compound engineering. Canonization is what turns vibes into compounds.
- [[Software Engineering at the Tipping Point]] — AI as 10× amplifier. Every node in the developer ecosystem breaks at 10×, including the canonization pipeline.
- [[The Joy and Power of Understanding]] — LLMs are force multipliers but you must have force first. Canonization requires understanding.
- [[Martin Fowler and Kent Beck on Reinventing Software]] — Two Agile Manifesto authors on AI's magnitude. The re-soloing illusion and why nobody has the answers anymore.
- [[Claude Is Not Your Architect]] — The attaboy problem and context-blind design. Canonization as the antidote to letting AI slide from assistant to architect.
- [[Writing Code vs. Shipping Code]] — 180% AI-driven commit gains attenuate to 30% at release. Disposable code is easy; shipping is hard.
- [[Discovery Debt]] — Accumulated untested assumptions that compound invisibly. The companion concept: discovery debt is what you don't know you don't know; absent canonization is what you know but haven't made reusable.
- [[The solution might be cancelling my AI subscription (Wilson)]] — 70 AI-built projects, none worth keeping. The overhang gets nothing from disposable code.

---
*Sources: [[raw/canonization-and-the-overhang]]*
*Last updated: 2026-07-08*
