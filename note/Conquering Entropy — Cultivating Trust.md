# Conquering Entropy — Cultivating Trust

kaeruct's practitioner prescription for the trust problem at the heart of AI-generated code. Rather than more tooling or better models, the fix is a *culture*: make engineers accountable for what they ship, give them the agency to use their preferred methods, and back the whole thing with deterministic tooling, human-defined tests, small PRs, and a taste for outcomes over code.

---

## Key Quotes

> "Most of the issues I have with AI-generated code are related to trust."

The thesis in one sentence. Notice what it does *not* blame: the model, the prompts, the harness. The trust chain is social and organizational — ticket author, PR author, test suite, CI/CD, observability, even GitHub's uptime — not technical. This is the same conclusion [[Agentic Code Review]] reaches from data (the bottleneck moved from writing code to trusting it), arrived at here by introspection rather than survey.

> "Trust is hard to earn and easy to lose. So I think it's crucial to foster a culture of trust within the engineering team. You must trust engineers to do the right thing."

The inversion worth sitting with: most "AI governance" writing is about *not trusting* the agent and *not trusting* the engineer. kaeruct's frame is the opposite — trust engineers, and spend your governance budget making that trust well-placed rather than making it unnecessary.

> "You are accountable for what you ship. If this breaks production and you authored the PR, you should be there to fix it."

Accountability is the load-bearing idea, and it's explicitly paired with agency in the next breath: if you're accountable, you must be allowed to choose your own methods and install your own quality measures. This is the team-level operationalization of [[Radical Accountability]]'s individual-level thesis — accountability as a *gift with conditions*, not a threat.

> "Even before coding agents most code was already a buggy mess! So it's understandable when a 'good enough' solution is delivered."

The deflationary move that separates this essay from the quality-panic genre. It refuses the false baseline of a pre-AI golden age. "Reasonable" quality, not 100% perfect, is the bar — which reframes technical debt as an accepted trade for speed, not a moral failure. This is the honesty that [[Agents and Acquiring Debt]] names as comprehension debt but here treats as *ordinary*, not AI-specific.

> "Make heavy use of deterministic tooling to ensure code quality. Typed languages, linters, dead code detection, security scans, CI/CD, and so on. All of these tools existed before coding agents and help us in the fight against slop."

The [[Guardrails and Feedback Loops]] thesis, compressed: linters beat prompts. The telling detail is "these tools existed before coding agents" — the fight against slop does not require inventing new machinery, just actually using the old deterministic kind.

> "Coding agents can implement the tests, but they should be defined by humans."

A precise division of labor that cuts through the "can you trust AI-generated tests?" debate. The human plays business analyst — think deeply about the feature, define test scenarios, discuss them with the team — and the agent does the mechanical implementation. This is [[Specifications as the Product]] applied one layer down: the *test scenario* is the durable artifact, the test code is disposable.

## Key Themes

#trust #engineering-culture #accountability #agentic-development #deterministic-tooling #pattern

The seven practices are a stack, not a list. **Coding guidelines** and **deterministic tooling** are the substrate (feed-forward and feedback controls, in [[Guardrails and Feedback Loops]]'s harness language). **Small PRs** and **human-owned tests** are the gates. **Taste in product**, **prototyping**, and **focus on the outcome** are the altitude — keeping the whole machine pointed at user value instead of code volume.

## Critical Analysis

The essay's strength is its altitude. It does not try to be a tooling survey or a data dump; it's an engineering-manager's manifesto, and it earns that register by keeping the argument short and the prescriptions concrete. "You are accountable for what you ship" is the kind of sentence you can paste into a team charter tomorrow.

Its quiet radicalism is the trust-first frame. The dominant genre right now is *distrust engineering* — [[Agentic Code Review]]'s "agents will weaken CI to make themselves pass," [[How We Contain Claude]]'s isolation patterns, [[The Enterprise Gap from Vibe Coding]]'s policy halts. kaeruct doesn't disagree with any of that; he just argues it's the wrong frame to *lead* with. Start from trusting engineers, then install the deterministic machinery that keeps that trust from being naive. It's a reframe more than a contradiction, and a useful one.

The gap is the same one [[Agentic Code Review]] names and [[Principal Drift]] fills: *how* do you sort changes by risk? kaeruct says "empower the engineers to reject unreviewable PRs" and "codify this criteria," but stops short of the criteria itself. That's fine for a manifesto — the risk model is a different essay — but a team that adopts the culture without the codified thresholds will find "reasonable quality" drifting toward "whatever the agent generated."

One claim deserves scrutiny: "you must trust engineers to do the right thing." [[Radical Accountability]]'s weakness section already notes that this requires both taste *and* the authority to act on it, which most engineers in most organizations lack. kaeruct gestures at this with "if engineers are made accountable... they should be given the agency" — but agency is granted by managers, and the essay doesn't address the organization that refuses to grant it. The prescription is correct; it's just not available everywhere.

---

*Sources: [[raw/conquering-entropy-cultivating-trust]], [[summary/conquering-entropy-cultivating-trust]]*
*Last updated: 2026-09-08*
