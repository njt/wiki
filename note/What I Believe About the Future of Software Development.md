# What I Believe About the Future of Software Development

Thorsten Ball's September 2026 listicle of seventeen predictions, planted as a
dated flag after the original X post "blew up": code review and unit tests die,
the craft of writing code disappears while the craft of building software
becomes more valuable, tokens replace the terminal as the computing paradigm,
the PM/Design/Eng triad dissolves, and access to tokens gates who gets to make
software at all.

---

## Key quotes

> **Code review will die.** I mean: it's already dead. But in the future, humans won't find a bug or an issue with the code produced by a model, at least not in a reasonable time. Humans will only review the system and its composition.

Ball's argument for why review dies is *capacity*, not policy: humans can't
keep up, so they'll review at the level of the system rather than the diff.

> **The craft of writing code will disappear.** Yes, there are still Italian shoe makers around. But look at your feet.

The essay's best line — scarcity of a craft and demand for it are decoupled.
Bespoke code survives like bespoke shoes: as a premium edge case.

> Most bugs won't be "coding" bugs. They'll be "you asked for the wrong thing" bugs.

This single claim does the most work in the piece: it's what makes the rest
follow. If correctness of intent replaces correctness of implementation, then
specification, not code, is the durable artifact.

> **The terminal is dead.** Most developer tooling will be washed away by tokens. (I'm saying this as a lover of the terminal & dev tools.)

Said with the self-awareness of a man who built books around the terminal and
editors. The comparison — knowing jq will feel like Perl one-liners — is the
concrete image that sticks.

> **Tokens are the new computing paradigm.** We've had deterministic computers for 80 years, so we confuse "how computers have worked" with "how computers must work." We're entering the post-binary era.

The most sweeping and least falsifiable claim. It's a frame, not a prediction:
everything else on the list is a corollary of it.

## Key themes

#concept — review shifts from diffs to systems and composition
#concept — "you asked for the wrong thing" bugs as the new failure mode
#pattern — craft (writing code) versus profession (building software)
#person — Thorsten Ball, terminal devotee predicting the terminal's death

## Analysis

This is a mood piece, and a good one: it's a maximalist position stated
plainly, from someone with the credibility of having written *Crafting
Interpreters* — the perfect person to declare the craft dead. That framing is
its strength and its weakness. Several predictions are falsifiable and dated
(review dies, tests die, terminal dies), several are unfalsifiable frames
("tokens are the new computing paradigm"), and the piece doesn't separate
them. The unit-test claim is the weakest: a model compiling 900 lines of
Arduino without error proves first-pass correctness on a small bounded
program, not that verification disappears as programs and intent grow. His
own admission that "you asked for the wrong thing" becomes the dominant bug
class implies *more* verification machinery, not less — just pointed at
behavior and intent rather than unit boundaries.

The strongest claims are the economic ones, which are already observable:
token access as the new gate on producing software, big-company process
looking sillier when constraints vanish, and "meat proxies" shoveling tickets
into agents losing their value. The piece pairs naturally with the wave of
dark-factory writing — it supplies the cultural manifesto those systems are
the engineering reality of.

## Relations

- Sharpens the thesis of [[The End of Code Review]]: where Monperrus argues
  mandatory human review is *indefensible*, Ball goes further — it's already
  dead, and what replaces it is review of system composition, not code.
- Complicates [[Agentic Code Review]]: Osmani's field guide documents a
  bottleneck *shifting* (reviews getting longer, churn rising); Ball predicts
  the bottleneck's *abolition*. The tension between those two readings is
  exactly what the next two years will adjudicate.
- Echoes [[Don't Fear the Dark Factory]] from the cultural side: if humans
  review systems and intent rather than code, the dark factory's
  comprehension-debt problem stops being a bug and becomes the plan.
- Its "you asked for the wrong thing" bugs give [[The Coming Need for Formal
  Specification]] its strongest motivation: if intent errors dominate,
  specification is where engineering effort must move.

---
*Sources: [[raw/what-i-believe-about-the-future-of-software-development]], [[summary/what-i-believe-about-the-future-of-software-development]]*
*Last updated: 2026-09-29*
