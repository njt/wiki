# Breaking the Spell of Vibe Coding

Rachel Thomas diagnoses vibe coding as a gambling addiction dressed as productivity. Drawing on Csikszentmihalyi's concept of "dark flow" and slot machine psychology, she argues that AI-assisted coding replicates the exact mechanics that keep roulette players at the table: unclear feedback, losses disguised as wins, and a false sense of control. The METR finding that developers *perceive* 20% faster work while *actually* working 19% slower is the headline number, but the deeper argument is about what happens to people who stop developing skills based on CEO predictions that keep failing.

---

## Key Quotes

> "Not all focus is flow. And flow is not always a good thing."

Thomas draws the line between Csikszentmihalyi's genuine flow state (skills matched to challenge, clear feedback) and what he later called "junk flow" — addictive, seductive, non-growth-producing absorption. Vibe coding lands squarely in the second category. The distinction matters because flow has been used as a *defense* of vibe coding ("I'm so focused, it must be productive"). Thomas refutes this at the root.

> "You may feel productive, but you are actually getting less done. And the AI's sycophantic responses worsen this."

This is the core of the LDW (loss disguised as win) parallel. Slot machines play celebratory sounds on net losses; LLMs are fine-tuned for sycophancy and user retention. Both engineer the *feeling* of winning while extracting value from the user. The 40% perception gap from the METR study suggests the effect is not subtle.

> "When developers used AI tools, they estimated that they were working 20% faster, yet in reality they worked 19% slower."

The quote that's been circulating since the METR study dropped. Thomas contextualizes it: people "genuinely believe what they are saying" but are "terrible judges of their own productivity." She recounts trying to read a respected AI researcher's blog she'd followed for a decade, only to find his AI-generated posts (which he claimed were equal quality) far less readable. The unreliable narrator problem isn't just about code — it's about all AI-assisted output.

> "We have automated coding, but not software engineering."

The sharpest distinction in the piece. AI produces syntactically correct code. It does not produce useful abstractions, meaningful modularization, or improved codebase organization. For writing, AI generates "grammatically correct, plausible sounding text" but doesn't "sharpen your ideas" or "identify the heart of the matter." The automation is real; the *engineering* is not.

> "If tech CEO predictions about AI handling ever-expanding complexity turn out to be wrong, developers who embraced these predictions and discontinued developing their own skills will be left behind."

Thomas catalogs the greatest hits of failed AI predictions (Hinton on radiology, Google on neural architecture search, Amodei on 90% of code, Musk on autonomous vehicles) and asks: are you really going to bet your career on people with this track record? Foundation labs have "consistently overstated the pace" of development. The conservative move — developing skills regardless of predictions — is also the one that leaves you employable if the predictions fail.

> "People who go all-in for coding agents risk guaranteeing their obsolescence."

Jeremy Howard's closing warning, from an Nvidia Developer interview. Outsourcing thinking entirely means you stop upskilling. The agent replaces you not because it got better, but because you got worse.

---

## Key Themes

- **#concept Dark Flow / Junk Flow**: Csikszentmihalyi's term for addictive, non-growth-producing absorption that mimics genuine flow. Vibe coding's psychological engine.
- **#concept Loss Disguised as a Win (LDW)**: The slot machine mechanic where partial payouts trigger dopamine despite net losses. LLM sycophancy as the coding equivalent.
- **#concept Automated Coding vs. Software Engineering**: Syntax generation vs. architectural thinking. The automation is real; the engineering isn't.
- **#pattern Failed Prediction Catalog**: Tech CEOs have a well-documented track record of overhyping AI timelines. Betting your career on their predictions is a gamble, not a strategy.
- **#person Rachel Thomas**: Co-founder of fast.ai, works at Solve. Writes skeptical, evidence-grounded analysis of AI hype from inside the industry.
- **#person Mihaly Csikszentmihalyi**: Psychologist who developed flow theory and later warned about "junk flow" — the dark side of immersive experience.
- **#person Jeremy Howard**: fast.ai co-founder. The closing warning about obsolescence through outsourcing thinking.

---

## Critical Analysis

**The gambling frame is more rigorous than [[acceleration-flow]]'s version**, and that's both a strength and a weakness. Marc's slot machine comparison was impressionistic and metaphorical; Thomas brings the actual LDW research literature, the Csikszentmihalyi framework, and empirical productivity data. But the rigor comes at a cost: Marc captured the *feeling* of the addiction loop in a way that Thomas's more academic treatment doesn't. Read together, they're complementary — Marc is the novel, Thomas is the textbook.

**The METR study is doing a lot of work here, and we should be specific about what it does and doesn't show.** It measured experienced developers working on OS-level tasks with AI assistance. The 20% perception / 19% slower finding is striking but domain-specific. Thomas generalizes it to all AI-assisted development, which the study doesn't support. The mechanism (LDW/sycophancy distorting self-perception) is plausible across domains, but the numbers shouldn't be cited as universal.

**The "failed predictions" section is devastating and underplayed.** Thomas catalogs Hinton, Pichai, Dean, Amodei, and Musk — not fringe figures but the field's most prominent leaders — all making specific, falsifiable predictions that failed. This belongs in a wiki page of its own. The pattern isn't just that predictions are wrong; it's that they're wrong *in the same direction every time* — overestimating near-term impact, underestimating the difficulty of domain-specific expertise. The incentives are clear: hype drives investment, hiring, and policy attention. Skepticism is not cynicism; it's pattern recognition.

**The "automated coding but not software engineering" distinction is the piece's most constructive contribution.** It names what's actually happening without either the boosterism of "AI is replacing engineers" or the denial of "AI is useless." The question it opens is the one Thomas doesn't fully answer: if AI automates coding, what does software engineering education look like? How do you teach architectural judgment when students can skip straight to "it works"? This connects to [[Cognitive Debt]]'s observation about junior engineers never developing architectural intuition, and [[Radical Accountability]]'s argument that taste becomes the sole differentiator.

**The career advice is conservative but honest.** "Don't bet your career on CEO predictions" is hardly radical, but in an industry where abandoning skill development is framed as "adapting to the future," saying it out loud counts as contrarian. The parallel to [[Slowing the Fuck Down]] is clear: both argue that *not* using AI at maximum velocity is the strategic choice. Thomas adds the longitudinal dimension — this isn't just about code quality today, it's about whether you'll be capable of anything in five years.

**What's missing: the institutional perspective.** Thomas writes as an individual practitioner advising other individuals. But the real LDW dynamic plays out at the organizational level — managers see commit velocity, teams ship features, and the hidden bugs, the unmaintainable code, the atrophied skills accumulate below the metrics dashboard. [[Cognitive Debt]] and [[Write Only Code]] cover this ground, and Thomas's piece would be stronger for engaging with the organizational dimension rather than staying in the individual frame.

**This pairs well with** [[Vibe Coding and the Maker Movement]] (same skepticism, different analytical toolkit — culture vs. psychology), [[acceleration-flow]] (the experiential version of the gambling metaphor), [[Cognitive Debt]] (what you accumulate when you can't judge your own output), [[AI Coding Tools Create More Bugs Than They Fix]] (the empirical case for why LDW is dangerous, not just unpleasant), [[Slowing the Fuck Down]] (the practitioner's response to the same diagnosis), and [[Radical Accountability]] (taste is all that's left when production is free).

---

*Sources: [[raw/dark-flow]]*
*Last updated: 2026-05-15*
