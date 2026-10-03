# Teaching AI to Work Like a Senior Engineer (Motta)

Jefferson S. Motta's six-month field report on the problem nobody writes skills for: making the agent understand *you*, not just your codebase. After mining his own conversation history for every time Claude got the tone wrong, he wrote `jefferson-senior-dev` — a skill that encodes rules of engagement for being treated as a technical peer rather than a beginner — plus a prescriptive prompting skill for Cursor, and watches his agents graduate from "context" to "posture calibration" to executable "units of work" fired across seven platforms.

---

## Key Quotes

> "The most persistent problem was posture rather than technical capability. The AI treated me like a beginner when I had 30+ years of experience behind me."

The whole article hangs on this reframing: the failure mode isn't model capability, it's the *default persona* — generic-cautious, option-offering, apologetic. Most skill-writing advice targets domain knowledge; Motta targets relational contract. This sharpens [[Tuning Claude Code Into a Better Engineering Partner]], which tunes configuration for workflow reliability but never asks the agent to model the human.

> "When I share code, assume it's correct until you find clear evidence otherwise. Don't question out of generic caution... If you find nothing wrong, say it's correct and move on. Don't invent caveats to look useful."

The core rule, and it's an inversion of the usual defensive posture: he explicitly buys the risk of un-questioned errors in exchange for signal. What makes it credible rather than sycophancy-bait is the two asymmetries he builds in — security issues are *always* flagged regardless, and his own known blind spot (swapped true/false booleans) is a standing flag order. "A good skill teaches AI when to disagree with you."

> "Declaring your own blind spots to an AI costs nothing, and few engineers do it. If you know where you fail and it knows too, you get a second check that never gets tired."

The best idea in the piece and the one least discussed elsewhere: self-declared error patterns as a first-class skill input. It also proves its worth — with false suspicions gone, the freed attention caught a discarded awaited call (fire-and-forget wearing the costume of disciplined async), "which is exactly why it survives human review: nobody reads past the await."

> "It became a CLI parameter... It ran on six of the seven platforms. The seventh didn't have the module involved and was correctly skipped. The build passed on all six."

The quiet structural pivot of the article: a skill evolves from documentation, to posture, to *a unit of work the pipeline can execute across N repositories*. With the insisted caveat: "a green build is not a validated fix" — end-to-end testing stays human. This is a concrete instance of the trajectory [[Teaching the Agent Our Craft]] describes, skills as encoded process rather than prose context.

> "Calibrated AI accelerates whatever you told it to do, including the wrong thing."

The honest counterweight. His own mistakes are logged without flinching: a broad cleanup instruction that replaced `??` with dashes across dozens of files, parallel pipeline sessions producing phantom failures, and a "flaky" build that CSV statistics revealed was deterministic (x86 vs AnyCPU). The skill fixes the AI's tone; nothing in the skill fixes his scoping. Measurement, not more calibration, was what caught the deterministic failure.

> "It's a Markdown file with the right rules, loaded in the right context. A skill is actionable documentation: it records what you know and where you accept being corrected."

Against the fine-tuning/RAG framing of "training" — the entire investment is one well-written file, and its definition of success is specifying the disagreement surface. Pairs naturally with [[Writing a Good CLAUDE.md]] and [[What You Bring to AI Determines the Result]]: the leverage is in what the human brings, deliberately encoded.

## Key Themes

#pattern #tool #workflow

### Posture calibration as a skill category
Not domain knowledge, not workflow scaffolding — a third kind of skill: rules of engagement between one human and one agent. Mined from reviewing conversation history, which is the right empirical method.

### Declaring your own failure modes
The inverted-boolean rule is the article's most original contribution: the human's known weaknesses as explicit, standing flag orders. Cheap, zero-embarrassment second-checking, and it concentrates the agent's scepticism where it pays.

### Skills become executable units
The arc from Part 1's context-file to a pipeline parameter firing a documented fix across six platforms — with green-build-is-not-validation kept honest.

### Speed raises the cost of your own mistakes
Calibration removed friction; friction removal amplified a bad scoping decision into dozens of corrupted files. The lesson lands as measurement (CSV stats), not more prompting.

## Opinionated Take

This is the best articulation I've seen of a real and under-documented practice: writing a skill *about yourself*. Most skill literature is project-shaped; Motta's is person-shaped, and the "when to disagree with me" framing exposes something true about agent review — generic caution is noise, targeted scepticism is signal, and only the human knows where the scepticism belongs. The weaknesses are worth naming too: the resume-skill approach is idiosyncratic and probably doesn't transfer without the weeks of conversation mining he did; "assume it's correct" trades a known small risk (his blind spot) against un-modelled unknown ones, which works for a 30-year veteran and would be malpractice advice for a junior. And the multi-repo skill-as-unit-of-work moment is genuinely novel but arrives half-argued — six green builds and an honest caveat, no end-to-end story yet. Part 3 apparently covers what broke when infrastructure lagged; the observability gap ("debugging in the dark, in 2026, is embarrassing") is the better-reviewed half of his month.

This source strengthens [[Tuning Claude Code Into a Better Engineering Partner]] by adding the missing dimension — that work tunes the *tool's configuration*, this tunes the *relationship*, and both are needed. It complicates [[Teaching the Agent Our Craft]]: Haldeman encodes team craft into skills; Motta encodes one person's trust contract and blind spots, a different and more personal granularity. And it echoes [[What You Bring to AI Determines the Result]] with concrete mechanics for *how* to bring yourself: write the failure patterns down.

---
*Sources: [[raw/6-months-claude-cursor-part-2-teaching-ai-work-senior-engineer]], [[summary/6-months-claude-cursor-part-2-teaching-ai-work-senior-engineer]]*
*Last updated: 2026-10-03*
