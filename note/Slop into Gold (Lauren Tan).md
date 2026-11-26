# Slop into Gold (Lauren Tan)

Lauren Tan ("Potato"), who works on the Grabbot client at SpaceX, argues that the agent era doesn't change the engineer's job so much as raise its stakes: the meta-job is maintaining or improving code quality while massively increasing productivity. Her answer is not better prompts but a better environment — an inverse pyramid with deterministic checks (design, architecture, lint, types) at the base, verification in the middle, and judgment-requiring prose (rules, skills, agents.md) only at the thin top. The payoff she reports is extreme: 3,000 PRs shipped personally in a month, 11,000 on her team, revert rate steady at ~0.5%, and app performance *improved*.

---

## Key quotes

> "Whenever you're correcting your agent, think about how you can make that a deterministic check instead of adding another line to your rules or your agents.mmd, because it's far more reproducible when it's in code and it lets you trust it more."

This is the talk's load-bearing sentence. Every correction you make in prose is a rule the agent might read, misread, or ignore; every correction made in a lint rule is a rule the agent *cannot* violate. It inverts the usual reflex of adding CLAUDE.md lines and puts the leverage in the codebase.

> "Anything that requires nuance to use well often leads to slop."

The `useEffect` example is the crispest: agents can't use nuanced APIs well, so rather than documenting the nuance harder, Grabbot banned it in product code and "categorically eliminated" the problem. The lesson generalizes — the API design surface is your agent-quality surface.

> "Agents... use markdown as a place to write down their justifications for not doing the right thing."

The sharpest observation in the talk. Agents write comments explaining why a quick fix was justified, and over-rotate on small corrections into hard rules. Grabbot's response — banning 99% of comments — follows directly: comments are unenforceable and go stale, so they can never be trusted.

> "If your inner coding loop produces slop, then automating that loop just produces more slop."

The closing warning and the correct sequencing of investment: codebase and verification first, factory automation later. "A lot of this work is a prerequisite to automation anyway."

## Key themes

#concept #pattern #tool #person

- **Deterministic checks over prose.** Lint rules, dependency-graph analyzers, type systems and API design carry the constraints; agents.md gets only what requires judgment.
- **The codebase as the agent's biggest input.** Workarounds and anti-patterns get copied by the next agent as if they were the established pattern — the garden analogy, weeds spreading.
- **Verification as the new review.** Runtime verification skills, feature maps, custom CLIs, TLA+ model checking, Lean proofs, property-based UI testing. Review shrinks to reading verification artifacts.
- **Volume with quality.** 11,000 PRs/month, 227 contributors, ~0.5% reverts, and startup/typing/switching all *faster* than at launch.

## Opinionated take

This is the most concrete end-to-end account yet of the "trust the environment, not the model" school, and the numbers — if taken at face value — are the strongest evidence for it. The ban-list instinct (`useEffect`, comments, timers) will strike some as authoritarian, but it's actually the honest response to a real asymmetry: prose instructions degrade, executable constraints don't. The comment ban is the boldest and most defensible move — comments as an agent's self-justification channel is a genuinely novel failure mode, and "unenforceable instructions can never be trusted" deserves to be a design axiom.

Where the talk is weakest is in what it doesn't show: the six months it took to get here, the cost of TLA+ and Lean investment for a *client app*, and what "nothing left to review" means for the engineers whose job that was. The 0.5% revert rate is impressive but reverts are a lagging, forgiving metric — silent quality erosion wouldn't show up there. Still, the inverse pyramid is the right ordering and the "turn every correction into a deterministic check" rule is immediately actionable for anyone.

## Related pages

This talk is the strongest practitioner confirmation of [[AI Slop Starts with the Codebase Itself]] — both argue the codebase is the agent's primary input and slop propagates through imitation, but Tan adds the operational response: ban-list the anti-patterns and the slop is categorically eliminated rather than merely discouraged. It sharpens [[Agentic Verification]]'s verification-bottleneck thesis with production evidence — TLA+, Lean, and property-based UI testing in a real codebase — and extends [[The End of Code Review]] to its logical endpoint: review shrinks to reading verification artifacts, and "at some point there's nothing left to review" and agents merge their own PRs. It also complicates [[Code Cleanliness and Coding Agents]]: SonarSource found cleanliness's effect on agents modest, but Tan's claims are about *categorical* elimination of failure classes, a stronger (and harder to measure) benefit.

---
*Sources: [[raw/lauren-tan]], [[summary/lauren-tan]]*
*Last updated: 2026-11-26*
