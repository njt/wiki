# Human-in-the-Loop is Tired

Laura Summers's field report on the psychological cost of LLM-assisted programming: AI makes coding faster and more intense but replaces the craft's small satisfactions with the cognitive drain of supervision. The piece names the "human reward function problem" — LLM-assisted work breaks the feedback loop that made programming fulfilling, replacing dopamine hits from problem-solving with the exhaustion of reviewing mostly-correct output.

---

## Key Quotes

> "The satisfying part shrank. The exhausting part grew."

This is Summers's most compressed statement of the core problem. The automation doesn't remove work — it shifts it from creation (rewarding) to supervision (draining). The ratio changes; the total doesn't.

> "The bottleneck was never the code. It was always the human attention, the engineering judgment, the ability to hold a coherent vision for a system."

The reframe that makes sense of the discomfort. Code generation was never the scarce resource. What's changing is that the scaffolding that made human judgment feel central is being stripped away, and what's left is judgment, exposed and unaugmented.

> "The humans are still in the loop. We're just tired."

The title drop and closer. Not a declaration of defeat — a diagnosis. The loop isn't going away; the work inside it has changed. Fatigue is the signal that the tooling has advanced faster than the practices for wielding it.

> "Programming with an LLM is an intensely solitary activity."

Natural collaboration points — turning to a colleague to talk through a problem — get replaced by another prompt. The Skinner Box of variable LLM rewards makes solo prompting more addictive than pair programming, even though it's less satisfying.

> "The craft didn't die, it evolved."

Summers is careful not to be a declinist. Her responsive-design analogy (the 2009 fixed-width-to-fluid transition) is the right historical parallel: designers experienced existential loss of control, then reframed their skills around systems thinking. The difference she acknowledges: this transition is measured in months, not years.

> "at that point, what am I still doing here?"

Colleague Douwe's question on waking to 30 overnight AI PRs. Not rhetorical — structural. When both the writing AND the reviewing can be delegated, what's the human for? Summers's answer: taste, architectural judgment, and the willingness to make contrarian calls.

---

## Key Themes

**#concept The Human Reward Function Problem** — Summers's central contribution. Hand-coding provides frequent, small dopamine hits (understanding logic, watching code compile, solving problems). LLM-assisted work replaces these with the cognitive load of review — holding intent in your head while evaluating output. The feedback loop that made programming satisfying is broken, and the fix isn't a better model; it's redesigning the workflow.

**#pattern Supervision Fatigue** — A new category of burnout distinct from overwork. It's not that you're writing too much code; it's that you're reviewing exponentially more code than anyone could write, and the review itself is harder (you didn't write it, so you have to reconstruct intent). Summers's two-day planning session yielding "errors of coherence" is the canonical example: smart enough to look plausible, not smart enough to maintain coherent intent.

**#pattern The Intensity Trap** — AI increases work *intensity*, not just speed. Variable rewards (Skinner Box) make "one more prompt" feel cheap, but each prompt generates output that demands review. You start more things; your brain is still the only thing that can finish them. Marcelo's joke about five simultaneous Claude sessions captures the trap: infinite parallelism of generation, zero parallelism of judgment.

**#concept Expertise Distillation** — The colleague who extracted rules from thousands of code review comments to seed an AGENTS.md is practicing something new: not writing code, not reviewing code, but *distilling* expertise into machine-consumable form. This is a legitimate new craft — it's just not the one anyone trained for.

**#pattern The Solitary Loop** — LLM-assisted work replaces human collaboration points with more prompting. The addictive quality (variable rewards, low friction) makes it self-reinforcing, but the outcome is isolation. Summers names this without offering a fix, which is honest — the structural incentives push toward solitude, and countering them requires deliberate effort.

---

## Critical Analysis

**What Summers gets right:** The reward function problem is real and under-discussed. The industry conversation about AI tools focuses on productivity metrics (PR count, cycle time) while ignoring the experiential cost. Summers's diagnosis — that the satisfying part shrank while the exhausting part grew — matches the lived experience of anyone who's spent a day reviewing AI output. This isn't resistance to change; it's an accurate description of a workflow that hasn't been designed for human psychology.

**What's missing:** The piece names the problem vividly but is thin on solutions. The responsive-design analogy feels more like reassurance than a roadmap. "Taste, nuance, mature architectural opinions" are real things that survive, but Summers doesn't address how they're developed when the craft ladder has been shortened — you can't develop mature architectural opinions without first writing a lot of bad code, and AI is absorbing that phase.

**The unspoken tension:** Summers works at Pydantic, a company building tools (Pydantic AI) that accelerate the very dynamic she's diagnosing. She acknowledges this in the closer ("We're debugging our reward functions in real time, same as you") but doesn't explore the structural conflict. This isn't a flaw in the essay — it's an honest admission that the people building the tools don't have answers either.

**The responsive-design parallel, examined:** Summers's 2009 analogy is the right historical reference but the wrong magnitude. Designers had years to adapt; the current shift is "measured in months." More importantly, responsive design changed how you *specify* layouts; it didn't automate the act of designing. LLM-assisted programming automates the act itself. The analogy holds for craft evolution; it breaks for pace and depth of change.

**Against the grain:** The most uncomfortable implication is Summers's observation that teams succeed most when guiding LLMs in domains they *already* deeply understand — and that in shallower areas, outputs become "impressionistic." If expertise is the prerequisite for effective AI use, then AI doesn't democratize programming; it stratifies it. The gap between experts (who can verify and steer) and novices (who can't tell plausible from correct) widens.

---

## Connections

- [[Dopamine Fracking]] — Summers's "human reward function problem" is the developer-specific instance of German S.'s broader diagnosis: optimization that depletes what it extracts
- [[Vibe Coding and the Maker Movement]] — The "evaluative anesthesia" concept is the other side of Summers's coin: the dopamine of making eclipses the ability to judge
- [[Lean Software Production]] — Matt Wynne's post-craft framework; Summers provides the visceral description of what post-craft *feels* like
- [[Automating Myself Out of Development]] — Nune Isabekyan's bottleneck shift from "no time to code" to "no time to review" is Summers's thesis lived in practice
- [[Agentic Code Review]] — Addy Osmani's metrics (861% churn, 441% longer reviews) are the quantitative backing for Summers's supervision fatigue
- [[Nicole Forsgren on AI and Developer Productivity]] — Why shipping hasn't gotten faster: the bottleneck shifted from inner loop to outer loop
- [[Writing Code vs. Shipping Code]] — Demirer et al.'s finding that 180% commit gains attenuate to 30% at release level; Summers explains the *why* — the human judgment bottleneck
- [[The Joy and Power of Understanding]] — Igor Roztropiński's argument that understanding is the intrinsic reward; Summers's piece is what happens when the reward is thinned
- [[The solution might be cancelling my AI subscription (Wilson)]] — David Wilson's friction-as-mechanism thesis is the personal answer to Summers's diagnosis
- [[Canonization and the Overhang]] — Kellan Elliott-McCrea's warning about rewarding only production depleting the cognitive seed corn; Summers's piece is a report from the depletion

---

*Sources: [[raw/human-in-the-loop-is-tired]]*
*Last updated: 2026-07-18*
