# LLMs Are Not Tools (Rich Hickey)

Rich Hickey — Clojure's author, a lifelong tool builder — argues that calling LLMs "tools" is a category error that hides what is actually being lost. A tool is something you *wield*: held in the hand, under continuous intention and direction, transmitting your force, building your expertise through use. LLMs are none of that; they are *automation* — "working by itself with little or no direct human control" — and mistaking one for the other means trading working with the world for working for a capricious robot.

---

## The argument

Hickey starts with definitions, deliberately. A tool is "a device or implement, especially one held in the hand" — and his preferred gloss is "a tool is something you wield": control, power, continuous directing of the course of events. Tool use "builds, rewards, and often requires, expertise and skill" — expertise from *experiri*, to try. Good tools are "predictable, reliable, deterministic, of bounded function, hopefully simple."

> "Tools are implements with which **you** do something, they do not do things for you."

> "LLMs are not tools with which you write code - they are automation with which you avoid writing code, to which you abdicate writing code."

The drive-thru is his anchor metaphor: it is "automation for avoiding cooking," not a tool for cooking. Prompt tweaking is ordering a custom burger (no onions) to game the automaton into giving you a fresh one — "filling **voodoo.md** with just the right incantations" — and "you are only a software update from everything you've 'learned' being rendered obsolete."

The deepest cost is epistemic. Delegating reading ("I just used it as a tool to read and summarize some research papers") means *it* read the papers, not you: "You miss learning how to write from those who can... You miss noticing things... You miss learning how to think." And he pre-empts the alignment framing: "Tools don't get 'misaligned', they don't 'go rogue'" — that vocabulary itself belongs to automation, not tools.

The gist closes with a reader's rebuttal (Aaron), who concedes the semantics but reframes: AI is "a functional foundry set" — tools machining tools — elevating the user "organizationally" into a director of tool-users. Hickey's published quip doesn't answer this; the question of whether director-of-automata is a skilled craft or an abdication with better branding is left hanging.

## Key themes

- #concept — wield vs. automate: control, feedback, and continuous direction as the defining properties of tool use
- #concept — expertise as a byproduct of wielding; automation deletes the learning, not just the labour
- #pattern — "voodoo.md": prompting as gaming a capricious automaton rather than directing a tool
- #person — Rich Hickey, arguing from a lifetime of tool building

## Critical take

This is the sharpest short statement of the craft-loss position in the current discourse, and its strength is definitional rather than empirical: once you accept "wield" as the criterion, the conclusion is analytic. That is also its weakness. The wield/automation dichotomy is a binary where most real use is a gradient — the same person who lets Claude summarize papers may hand-write the hard kernel of a system, and Hickey offers no vocabulary for the middle. His own "the LLM didn't write my paper" example is a strawman target he erects and knocks down.

Still, the essay lands because it names the thing practitioners confess privately: the shift from directing to divining. The voodoo.md image anticipates the prompt-superstition and context-incantation culture documented across this wiki, and the drive-thru metaphor is genuinely clarifying — most "AI tools" marketing is exactly a claim that avoidance-automation is wielding. Aaron's foundry reply is the serious counter: if tools-that-machine-tools is real, Hickey's hand-held definition may be parochial to the pre-composition era. But note that Hickey could concede Aaron's frame and still win the epistemic point — the director of a factory of automata who cannot weld has lost something specific, and the essay says precisely what.

## Relation to the wiki

- [[Understand to Participate]] — Litt argues understanding is the prerequisite for staying an active collaborator; Hickey supplies the sharper claim that abdicated understanding is *gone*, not deferred — automation doesn't hand the learning back later.
- [[The Joy and Power of Understanding]] — Roztropiński's "force multipliers but you must have force first" is the constructive mirror of Hickey's warning; both locate the danger in skipping the struggle that builds expertise.
- [[Breaking the Spell of Vibe Coding]] — Hickey's drive-thru framing gives the philosophical ground beneath that essay's practical critique: vibe coding is abdication rebranded as wielding.
- [[A Non-Anthropomorphized View of LLMs]] — shares the deflationary project of stripping comforting framing off LLMs, but replaces Hickey's tool/automation binary with a functional, non-metaphorical account.

---
*Sources: [[raw/619d60dea942a8d4498cba3bb6b11c02]], [[summary/619d60dea942a8d4498cba3bb6b11c02]]*
*Last updated: 2026-10-13*
