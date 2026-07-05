# The Cult of Vibe Coding Is Insane

Bram Cohen (creator of BitTorrent) takes a flamethrower to the extremist wing of vibe coding — the ideology that developers should deliberately refuse to look at or contribute to AI-generated code. Writing in response to a leaked Claude source dump, he argues the emperor has no clothes: "pure vibe coding is a myth," the infrastructure that makes AI coding work (plans, skills, rules) is human-built, and "bad software is a decision you make."

---

## Key Quotes

> "Bad software is a decision you make."

Cohen's thesis in six words. He's not arguing AI forces low quality — he's arguing the refusal to inspect output is an active choice with predictable consequences. This is the steelman version of [[AI Coding Tools Create More Bugs Than They Fix]]: the bugs aren't the AI's fault, they're the developer's.

> "Pure vibe coding is a myth."

The core of his technical argument. Even in the most hands-off workflows, humans provide the framework — plan files, CLAUDE.md, skills, hooks, rules. The machine "works very poorly without being given a framework." This converges with [[Harness Engineering]]'s argument that the scaffold matters more than the model, but Cohen's framing is sharper: claiming otherwise is ideology, not engineering.

> "Looking under the hood is cheating."

Cohen's sarcastic summary of the vibe-coding taboo. He finds it absurd that Claude's team apparently refused to inspect their own codebase for duplication — code "written in English" that "anyone could read." The refusal to look isn't a technical constraint; it's a purity ritual.

> "Projects are born in sin."

On technical debt: both AI-generated and human-generated codebases accumulate it. The difference is that AI is actually good at cleanup — Cohen describes his own workflow of initiating conversations about ugly functions, discussing the problem with the AI until both sides converge, then directing the AI to plan and build. Refactors that used to take a year now take "sometimes a matter of weeks."

---

## Key Themes

- **#concept Pure Vibe Coding** — The extremist version: make "literally no contribution to what's going on under the hood." Cohen argues this never actually existed; the framework-building work is real engineering and it's what makes the difference.
- **#pattern Collaborative Refactoring** — Cohen's personal workflow: start a conversation about a problem ("this function makes my eyes bleed"), discuss with the AI until convergence, then direct the AI to plan and build. The result looks like one-shotting but involves substantial back-and-forth.
- **#person Bram Cohen** — Creator of BitTorrent. Pragmatist voice in the AI coding discourse. Argues from hard-won software experience rather than ideology.
- **#concept Framework as Engineering** — The plan files, skills, hooks, and rules that vibe coders claim are "not coding" are actually the highest-leverage coding there is.

---

## Critical Analysis

**Cohen is right about the extremists and wrong about the mainstream.** The "never look at code" position is genuinely held by almost nobody serious. Most practitioners who call themselves vibe coders mean something closer to Cohen's own workflow: high-level guidance, AI handles implementation, human reviews judgment calls. He's torching a straw man, but it's a straw man that generates a lot of engagement bait.

**His refactoring claim is the most underdeveloped and most interesting part.** The idea that AI is *better* at cleanup than at greenfield generation — because cleanup is bounded by existing code structure — deserves its own essay. Most AI coding discourse focuses on generation speed. Cohen flips it: AI's real superpower might be the refactors that historically consumed whole quarters.

**The "infrastructure" argument converges with the whole wiki.** [[Harness Engineering]], [[claude-ctrl]], [[Feedback Loop is All You Need]], [[Guardrails and Feedback Loops]] — they all argue that the human-built scaffold is what makes AI output trustworthy. Cohen's contribution is the bluntest formulation: if you're not building that scaffold, you're not doing engineering, you're doing performance art.

**The BitTorrent pedigree matters.** Cohen shipped one of the most consequential peer-to-peer protocols in history. When he says "bad software is a decision you make," he's speaking from a career of making good decisions at scale. This isn't a thought leader guessing; it's a builder who knows what quality costs.

**What's missing:** Cohen doesn't engage with *why* the purity taboo formed. [[Vibe Coding and the Maker Movement]] has the answer: evaluative anesthesia — the dopamine of making eclipses the ability to judge. Cohen treats the ideology as irrational and moves on, but the psychology that sustains it is worth understanding.

This pairs well with [[Vibe Coding and the Maker Movement]] (the cultural analysis Cohen skips), [[Radical Accountability]] (taste as the scarce resource), [[AI Zealotry]] (senior engineers should lead, not abstain), [[Write Only Code]] (the extreme end of not-looking), [[Cognitive Debt]] (what you accumulate when you refuse to inspect), and [[Slowing the Fuck Down]] (deliberate friction as engineering practice).

---
*Sources: [[summary/the-cult-of-vibe-coding-is-insane]]*
*Last updated: 2026-05-15*
