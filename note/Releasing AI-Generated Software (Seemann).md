# Releasing AI-Generated Software (Seemann)

Mark Seemann takes Robert C. Martin's "I don't read the code my agents write" tweet seriously and asks two questions the rest of the wiki mostly ducks: if nobody reads the code, what makes software engineering still *engineering* — and when the AI-generated defect is serious enough, who goes to prison?

---

## The setup

Uncle Bob's strategy: never read agent code, surround the agents with "extreme constraints" — unit tests, gherkin tests, mutation testing, coverage, QA procedures. Confidence comes from the gauntlet, not from reading. Seemann notes Martin has since walked back on his harness use, but declines the exegesis; the tweet is a premise, not the subject.

## Reading code was never the safety net

Seemann's reflex — "if you're no longer reading the code, how do you know it's correct?" — lasts one second before he notices it's an old programmer's knee-jerk. Human code review catches some bugs and misses others; TDD's most common failure mode is the test you don't write, leaving the system under-specified without anyone noticing. Agent "teams" using different heuristics aren't obviously worse than the human processes we already trusted imperfectly.

> "Demanding perfection from LLM-based software engineering sets a bar we human programmers never cleared. Rather, we seem to arrive at the same argument that you often encounter related to self-driving cars: The system doesn't have to be perfect: It just has to be *better* than the human-centred system."

This is the self-driving-car bargain imported into software. Seemann's claim is modest: no *inherent* barrier to a fully LLM-driven release process, not that we've arrived.

## The liability gap

The interesting move is to run the thought experiment to its end. Defect rate lower than human — not zero. Most errors benign (a misaligned button, a wrong discount). Some are not: the lost spacecraft, the radiation overdose. And then:

> "But, your honour, the LLM wrote the bug."
> "So you say, son, but you signed off on the release, didn't you?"
> "I can't send an LLM to prison, but I *can* sentence you."

Courts will assign responsibility by their own logic, decoupled from how the software was actually produced. The sign-off is the liability surface, whoever or whatever did the writing.

The Tesla FSD analogy is the sharpest: Danish regulators approved FSD with the human legally responsible. Prediction — drivers become inured, zone out, and the rare accident becomes involuntary manslaughter for the person in the seat. Legislation lags technological reality, and people will "play the odds" because the system looks so compelling. Seemann expects the same dynamics in software engineering: elaborate verification doesn't shield you, because liability attaches to the human decision to release, not to the artifacts.

## Critical take

This is a genuinely different cut of the dark-factory debate. The wiki's factory literature ([[Software Factories, Light and Dark]], [[Don't Fear the Dark Factory]]) asks whether no-human-reads engineering *works*. Seemann asks who *answers* for it — a question measurement-heavy treatments leave entirely out. His epistemic honesty about human fallibility is welcome: most "someone must read the code" arguments smuggle in a nostalgic view of review that the evidence never supported.

Two weaknesses. First, the walk-back he dismisses matters more than he allows: if Uncle Bob abandoned the constraints-only strategy, that's evidence about whether the gauntlet sustains confidence in practice. Second, the liability argument cuts in favor of *more* human involvement, not less — if signing the release is what exposes you, understanding what you signed becomes rational self-interest, which is an economic argument for keeping humans in the loop that Seemann leaves implicit.

## Key themes

#concept — liability as the ungoverned variable of AI-generated software
#concept — self-driving-car standard: better than the human baseline, not perfect
#person — Robert C. Martin as the prompting premise
#pattern — constraints gauntlet replacing code reading

## Related pages

- [[Coding Is Not Solved — Alex Ewerlöf]] makes accountability the irreducible human cost and argues AI can't be held accountable — Seemann supplies the juridical mechanism Ewerlöf's ownership formula gestures at, and complicates it: accountability can't be assigned to the non-person who wrote the code, so it lands on whoever signed.
- [[Software Factories, Light and Dark]] catalogs the dark-factory comprehension-debt problem; Seemann strengthens its risk side with a new category — legal liability — that no test suite discharges.
- [[Own the Outer Loop]] locates human judgment in the outer loop where the Verdict gets issued; Seemann shows why that loop is now also the legal exposure point — evidence licenses the verdict, and the verdict is what a court reads.
- [[The End of Code Review]] argues mandatory human review is indefensible on throughput grounds; Seemann is the counterweight — the review-less future may be technically sound but leaves the release sign-off holding all the risk with none of the understanding.

---
*Sources: [[raw/releasing-ai-generated-software]], [[summary/releasing-ai-generated-software]]*
*Last updated: 2026-10-14*
