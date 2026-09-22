# AI Made Me Doubt Everything About Programming

Felienne Hermans's DDD Europe 2026 keynote is a breakup letter to computer science, delivered by someone with the standing to write one: professor, high-school CS teacher, and author of the Hedy teaching language. Her case is that the field's value system is broken — it pays difficulty and dismisses accessibility, whether the artifact is spreadsheets ("not real programming"), localized languages ("why don't they just learn English?"), or DDD ("talking to customers" dressed up as "event storming") — and that this culture, having filtered out everyone who cared about social change, is now handing the keys to LLMs without asking what programming is *for*. Her alternative is chess: machines solved chess in 1997 and chess simply barred them. Adopting LLMs for programming is a choice, and we could choose otherwise.

---

## Key Quotes

> "Recently I totally fell out of love with computer science. I don't think I like this field. I don't think I like what we're doing and the people in it."

The opening, and the talk never walks it back. The confession does real work: her credentials (two decades in, Hedy, the spreadsheet research) rule out the easy "sour grapes" reading, and the personal register is the argument — this is what the culture's gatekeeping does to the people it claims to want.

> "That's not real programming."

The response to her spreadsheet research, sustained for four years — *even after she demonstrated a Turing machine in spreadsheet formulas*. Her archaeology of the exchange is the sharpest part of the diagnosis: "real" was never defined, because it wasn't a criterion. It was a boundary-policing instinct, and the gate moved every time she satisfied it. "Why was that hard?" — the question Hedy got at a language design conference — is the same instinct pointed at worth: if it was easy to build, it can't have been valuable. Her rebuttal is the talk's moral center: even if implementing all 71 Hedy languages had taken five minutes, the teacher in Botswana who can now show students that "our language, Setswana, is also the language of technology" would still justify it.

> "You had a thing that 300 million people use, it is an invalid character."

On Eastern Arabic numerals. She tried the whole TIOBE top ten; only SQL accepted them. A perfect miniature of the accessibility critique — the failure isn't an engineering constraint, it's that nobody in the language community considered 300 million non-Latin-digit users worth a thought.

> "In 10 years a computer will be the world champion in chess — unless it is barred from competition." — Herbert Simon, 1956

The "unless" is the hinge of the talk. Simon predicted not just the capability but society's *response* to it — and society obliged: engines have beaten world champions since 1997, and competitive chess still bans them. Hermans's move is to treat this as the template: "solved" does not have to mean "adopted," and LLM adoption is a decision someone is making, not weather.

> "Maybe we have artificial intelligence, but certainly we don't have artificial intellectual activity."

Via Peter Naur's 1984 distinction: intellectual activity isn't doing the thing, it's having a theory — the knowledge needed "to explain them, to answer queries about them, to argue about them." Ask an LLM "why did you implement it like this?" and, in her telling, it says "because it was a Tuesday"; correct it to Wednesday and it apologizes and agrees. No consistent internal model, nothing to answer for. This is the strongest philosophical objection to "AI can code" because it refuses the capability framing entirely: the question isn't whether the output works, it's who can stand behind it.

> "Programming is to make programmers happy and programming is to make programmers proud. That's what it's for."

The indictment landing. A 2017 study she cites found the biggest predictor for *not* studying computer science is caring about social change. The field selected for complexity lovers, then organized itself around their satisfaction — "everything must be made more complicated and more interesting so that we have a more interesting life." Her evidence is the Wikipedia article on programming, which classifies analyzing requirements, testing, and debugging as "auxiliary tasks": the parts that face people are, definitionally, not the core.

## Key Themes

- #concept **The value system of difficulty** — feminist epistemology as a field diagnostic. The 2016 glaciers paper (heroic mountain glaciers oversampled, village-adjacent ones neglected) explains why Haskell and C are the mountains and spreadsheets and PHP are the rural glaciers.
- #concept **Intellectual activity vs. output** — Naur's theory-building as the human residue that LLMs don't produce.
- #concept **Chess as precedent** — Ensmenger's "Drosophila of AI": binary benchmarking leaked from chess into language and art; and the chess community's refusal of engine assistance as proof that a solved problem can stay unsanctioned.
- #concept **The unreconciled history** — von Neumann's detonation calculations and the IEEE medal that still bears his name; IBM's punch-card business and the 1937 medal from Hitler. Presented as the field's origin story surfacing again as AI in warfare.
- #tool **Hedy** — hedy.org; free, open-source, Python-like, localized into 71 languages including Arabic with Eastern Arabic numerals.
- #person **Felienne Hermans**; with Herbert Simon, Peter Naur, Nathan Ensmenger, Ada Lovelace, and David Graeber as her witnesses.

## Opinionated Take

**What lands.** The Naur argument is the best thing here — a 40-year-old philosophical framework that makes the LLM question about *accountability of understanding* rather than correctness of output, and it gives the practical arguments for keeping humans in the loop a foundation they usually lack. The Simon "unless" quote is a genuine archival find, and the chess precedent is a rare, bracing counter to the inevitability framing that saturates this wiki's agent-coding sources. The spreadsheet wars material is meticulously unfair to the field in a way that only twenty years of receipts can license.

**What doesn't.** The DDD swipe is cheap, and the talk's own omissions say so: "talking to customers wrapped in complexity" ignores bounded contexts, model integrity across teams, and strategic design at scale. There is an irony worth naming — Hermans is doing to DDD exactly what the language community did to spreadsheets: dismissing a practice because its packaging looks easy. The chess analogy also carries less weight than she needs: nobody's employer mandates chess engines, but programmers "choose not to use" LLMs inside firms that measure output — an economic coercion the talk names nowhere. And "programming is to make programmers happy" is a devastating line that her own career refutes: Hedy exists because she cares about 12-year-olds in Botswana, not complexity. The history section is presented as exposé but stops at "we need to talk about that history" — no ask, no mechanism, no target. The talk diagnoses a culture and then ends on inspiration: Graeber's "the world is something that we make" as a closing card rather than a strategy. What she calls hope, a skeptic would call the absence of one.

**The transcript caveat.** The gist's auto-captions garble the names — Hedy as "Haddy" and "hadi," Ensmenger as "Ensmanger," Naur as "Nauer," and Hermans's Dutch idiom ("standing next to my shoes of proudness") beyond recognition. The gist's summary file is the authoritative spelling; the transcript is the voice. Her position is also less anti-LLM than the room wants: "I don't necessarily have something against all of the technology" — her Lovelace quote ("its province is to assist us in making available what we are already acquainted with") sketches a legitimate role she never develops into criteria.

## Relates To

- [[Understand to Participate]] — Litt argues you must keep understanding your codebase to remain a participant rather than a rubber stamp; Hermans supplies the metaphysical foundation that argument has been missing (Naur: theory-building is what the work *is*) and then sharpens it into a harder question — if machines can't build theories and humans stop doing so, the collaboration Litt wants to preserve may not be worth preserving.
- [[DDD Matters More When AI Writes Your Code]] — Smółka's defense of DDD in the agent era and Hermans's attack on it pass through the same door: both agree the value is the team's shared understanding, not any artifact. Her "talking to customers, wrapped in complexity to be palatable" complicates his case by naming the accessibility critique DDD must survive — at scale and in its jargon, not just in its intent.
- [[Vibe Coding as a Team Sport]] — Udell cites Kasparov to argue "weak human + machine + better process beats either alone"; Hermans cites the same Deep Blue moment for the opposite lesson: chess chose to bar the machine. Same evidence, opposite conclusions, and the honest reading is that both moves are available — which is quietly her whole point about choice.
- [[Building When It Feels Like There's Nothing Left to Build]] — Huyen's answer to the AI meaning-crisis is to build for joy and local human preference; Hermans turns "building for joy" into the indictment itself — a field that builds for its own satisfaction filtered out everyone who cared about social change. The same existential question, answered in opposite moods.

*In passing:* her account of supervision replacing creation gives [[Human-in-the-Loop is Tired]] its philosophical shadow — Summers's "the satisfying part shrank, the exhausting part grew" is the felt experience of a field that traded theory-building for output.

---
*Sources: [[raw/ai-made-me-doubt-everything-about-programming]], [[summary/ai-made-me-doubt-everything-about-programming]]*
*Last updated: 2026-09-13*
