# Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper

Codemanship's three-tier rule for picking the right tool in an agent-saturated world: prefer deterministic tooling (IDEs, compilers, linters) wherever it exists, accept probabilistic agents only where deterministic tools can't do the job or doing it by hand is slower, and remember the human is also on the menu when they're the cheapest reliable option. Framed as an indictment of teams who invert this by maximising agent usage first.

---

The essay's inescapable conclusion is that "the AI thing isn't about productivity" — if it were, teams would optimise outcomes. Instead it keeps seeing teams whose goal is to use AI coding agents as much as possible, with impact a secondary (or "thirdary") concern: renaming a class via Claude Code when an editor shortcut does it with a fraction of the compute and more reliably; asking Copilot to find unused code the background compiler already found; explaining desired code to Codex in more words than the code itself, "and the model still gets it wrong."

The contrast is teams who use the best tool for the job: IntelliJ/Rider/PyCharm for most refactorings, the IDE and linters for detection, and plain typing when you know what code you need. The agent earns its place only at the gap — e.g. moving an instance method in Python, which PyCharm can't refactor — where it succeeds ~90% of the time and beats doing it by hand. "So they make a *rational choice* to throw the dice." Those are the teams whose lead times and release cycles actually shrank without costing reliability.

---

## Key quotes

> I've seen dev teams setting out with the specific goal to use AI coding agents as much as possible, with the impact a secondary (or even a thirdary) concern.

The target of the piece isn't agents, it's adoption-as-goal. Tool maximisation is a means that ate its own end — the same inversion [[The AI Productivity Paradox]] diagnoses at product level.

> And – 90% of the time – the model will do what they need. (And, annoyingly, sometimes more than they need.)

The parenthetical is quietly the most important line: the agent's failure mode here isn't failure, it's unrequested success. Probabilistic tooling costs you review effort even when it "works" — sometimes more than when it doesn't.

> If you know what code you need, maybe just write it. M'kay?

The rule's third tier, stated with deliberate anti-hype bluntness. The human is a tool in the toolkit, chosen when cheaper — which inverts the guilt narrative that typing code by hand now signals Luddism.

## Key themes

#concept #pattern #comparison

## Analysis

The headline rule is a decision procedure, and it's better than the usual "should we use AI?" framing because it's about cost classes, not capability classes. Deterministic tools have near-zero marginal cost and ~100% reliability; agents have token cost and a 90% hit rate plus a "sometimes more than you need" tax; humans have the most expensive input but perfect intent alignment. Route each task to the cheapest tier that's reliable enough, and most of the industry's agent-for-everything pathologies dissolve.

What the essay compresses: the tiers interact. The 90% success rate for "move the method" is only rational because a diff review is cheap for a small, well-specified refactoring — the dice-throw is priced by the verification cost behind it. That's the real structure of the rule: *probabilistic when necessary **and cheap to check***. Under-specify the check and the rational throw becomes reckless, which is the failure mode documented across the wiki's review-and-verification pages.

The essay also implies, but doesn't say, that the tiers are a moving frontier: PyCharm lacking a Python instance-method move is exactly the kind of gap that deterministic tools close over time. Tier assignment is a maintenance problem, not a one-time doctrine.

## Related pages

- [[The AI Productivity Paradox]] — strengthens it: Cagan's "output vs outcomes" split is the same inversion, diagnosed at product-model level where this essay catches it at keystroke level.
- [[Removing the Noise for Better Prompts]] — nuance from the same doctrine: Muse explicitly assigns dependency maintenance to deterministic tools, not agents — the rule already operating as skill-design practice.
- [[A Convention Is Not a Constraint]] — complicates it: conventions read by agents are probabilistic; constraints enforced by machines are deterministic, and this rule explains which tier each belongs in.
- [[Code Standards That Survive the Next Agent]] — the same tiering applied to standards: AGENTS.md for what agents interpret (probabilistic), CI for what machines enforce (deterministic).

---
*Sources: [[raw/deterministic-when-possible-probabilistic-when-necessary-human-when-cheaper]], [[summary/deterministic-when-possible-probabilistic-when-necessary-human-when-cheaper]]*
*Last updated: 2026-10-03*
