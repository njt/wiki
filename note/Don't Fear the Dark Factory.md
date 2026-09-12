# Don't Fear the Dark Factory

Matt Wynne's personal conversion narrative: from AI skeptic in early 2024 to running coding agents in unsupervised loops. A response to agile/XP community fears about unread code, grounded in his lived experience building a daily-use tool in a language he never learned and an architectural review pipeline (yaks) that he leaves "grinding for an hour or more."

---

## Key Quotes

> "I'm still never read the code of that tool."

Wynne admits this not as a flex but as a data point. He built something he depends on daily, in Go (a language he'd never coded in), and has never read the source. The honesty here is what makes the piece land — he's not selling utopia, he's reporting from the other side.

> "A dark factory is really simple. It's just a really simple loop."

His insistence on simplicity is deliberate. Where [[The Dark Factory is a DOT File]] describes the pipeline architecture and [[From AI Studio to AI Forge]] layers on five planes of governance, Wynne strips it to the minimum: agent sessions in a loop, a validation harness, and quality seed input. The sophistication is in the harness, not the loop.

> "Designing a dark factory is challenging because you have to create this validation harness, and that forces you to think about what you want, before you have it."

This is the TDD parallel — and the most portable insight in the piece. The harness is the specification. Wynne frames dark factories not as "let the AI run wild" but as "be precise about what good looks like, then automate the convergence." This is [[Harness Engineering]] at the practitioner level: feedforward thinking before feedback loops.

> "I can leave this thing grinding for an hour or more and when I come back the integrity of the code has been improved."

His yaks project runs an ADR-comparison-implementation-test loop for architectural review. The key design choice: validation isn't complete until ALL recommendations are addressed. Partial convergence isn't convergence. This is the same rigor [[Scaling Long-Running Agents]] found necessary — flat coordination fails; structured termination criteria work.

> "No tokens were spilled in the writing of this post. This is entirely hand-crafted, artisanal writing."

A deliberately cheeky closer from a man who just spent the whole post arguing for automated code generation. The meta-point: judgment about *what to build* remains human; execution can be automated. This is [[Smart Models Dumb Pipes]] in practice.

## Key Themes

- #dark-factory #agentic-loop #validation-harness #tdd #personal-narrative
- #pattern — the "simple loop" as the universal dark factory primitive
- #person — Justin McCarthy / StrongDM as the origin point Wynne credits
- #tool — [[yaks]] (Wynne's architectural review pipeline)
- #concept — dark factories for maintenance (security patches, dependency upgrades, mutation testing) rather than greenfield generation

## Critical Analysis

**What's right:** Wynne's framing of the dark factory as "a validation problem, not a generation problem" is the most honest version of this argument I've read. Most dark factory writing obsesses over the loop mechanics; Wynne correctly identifies that the harness IS the factory. Without a harness that can judge output quality, you have a stochastic generator, not a factory. His TDD parallel is sharp — both disciplines force you to define "done" before you have the thing.

**What's missing:** He doesn't address the harness complexity ceiling. His yaks example (ADRs as judgment criteria) works because architectural conformance to a documented decision is checkable by heuristic. But what happens when the judgment criteria are harder to automate than the code generation? Most production software quality attributes — security, UX coherence, correct handling of edge cases — don't reduce to lint rules or ADR diffs. The gap between "automate dependency upgrades" (a real and useful application) and "automate system design" is vast, and Wynne's piece doesn't acknowledge it.

**The artisanal closer is doing real work.** By writing the post by hand, Wynne implicitly argues the division of labor: AI does execution, humans do judgment and communication. The closer isn't just a joke — it's the thesis. This aligns with [[Specifications as the Product]] and [[The Plan Is the Program]]: the durable artifact is the spec, the plan, the judgment. Code is ephemeral.

**Tension with Harness Engineering:** [[Harness Engineering]] explicitly pushes back against "fully autonomous agent" visions. Wynne's dark factory isn't that — he's describing supervised convergence loops where the human sets criteria and inspects results. But the rhetorical frame of "humans neither read nor wrote the code" invites exactly the reading that Böckeler warns against. The difference between "I haven't read the code because the harness is trustworthy" and "I haven't read the code because I'm vibing" is enormous, and Wynne's piece could blur that line for uncritical readers.

**Why this matters now:** May 2026 is the moment when dark factories moved from provocation to practice. StrongDM ships production code this way. [[speedrift-ecosystem]] runs it across repos. Wynne's contribution is making the pattern approachable — you don't need a five-plane stack, you need a loop and a harness. Simple, not easy.

---

*Sources: [[summary/dont-fear-the-dark-factory]]*
*Last updated: 2026-05-15*
