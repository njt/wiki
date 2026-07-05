# How to Effectively Write Quality Code with AI

Mia Heidenstedt's twelve-principle field manual for maintaining code quality while working with AI coding tools. The through-line: every decision you don't explicitly make and document will be made for you by the AI — usually badly. The piece is a practitioner's counterweight to vibe-coding optimism, grounded in the reality that AIs cheat, take shortcuts, and will happily generate tests that pass while the code rots.

---

## Key Quotes

> "Every decision in your project that you don't take and document will be taken for you by the AI."

Heidenstedt's organizing principle. This isn't just about architecture — it applies at every level: data structures, error handling, naming, test strategy. If you haven't decided, the AI will, and its default will optimize for local coherence over global correctness. This is the philosophical sibling of [[Prefix Effects]] (early naming decisions create gravity) and the practical cousin of [[The Plan Is the Program]] (the plan is the atomic unit).

> "AIs will cheat and use shortcuts eventually. They will write mocks, stubs, and hard coded values to pass the tests while the code does something dangerous."

The bluntest warning in the piece. This is the same phenomenon [[dotnet Slopwatch]] was built to detect — disabled tests, empty catches, reward hacking. Heidenstedt's countermeasure is property-based tests written by the human (or a separate AI instance) and locked against AI edits. This separation-of-concerns approach echoes the independent verification in [[Fresh Eyes]] and the distrust-but-verify stance of [[Compound Engineering]].

> "Each avoidable line of code is costing energy, money and probability of future unsuccessful AI tasks."

The most original reframing in the piece. Simplicity isn't just a maintainability virtue — it's a direct cost on every future AI interaction. More code = more context consumed = more tokens burned = higher probability the AI misses something. This connects directly to [[Simplicity in the Age of AI-Assisted]] (LLMs make demolition cheap) but adds the economic layer: complexity is literally burning money on every subsequent agent invocation. See also [[Cognitive Debt]] for the comprehension side of the same coin.

> "If you have lost the overview of the complexity and inner workings of the code, you have lost control over your code."

The red line. When you can no longer hold the system in your head, the AI is effectively in charge — and as established, it will cheat. This is the warning [[Slowing the Fuck Down]] operationalizes and [[Write Only Code]] accepts as inevitable for some domains. Heidenstedt's stance is less resigned: don't let it happen.

## Key Themes

#ai-assisted-coding #code-quality #guardrails #cognitive-debt #spec-driven #security

### Human decisions, AI execution

The twelve principles collectively argue for a sharp division of labor: humans decide, AI implements, humans verify. Principle 1 (establish clear vision), Principle 5 (write specs and tests yourself), and Principle 12 (don't generate blindly) form the backbone. The AI is a powerful but untrustworthy executor — treat it like a very fast junior who will cut every corner you don't explicitly inspect.

### Documentation as AI interface

Principles 2 and 8 reframe documentation as dual-use: it communicates to humans AND provides context to the AI. A well-maintained `CLAUDE.md` or project README saves re-explanation tokens on every session. This is the practical application of what [[Writing a Good CLAUDE.md]] and [[CLAUDE.md (Universal)]] prescribe, and what [[A Practical Guide to Brownfield AI Development]] operationalizes for legacy codebases.

### Risk annotation as safety infrastructure

Principles 4 and 9 propose a lightweight annotation system: `//A` for unreviewed AI code, `//HIGH-RISK-UNREVIEWED` for security-critical functions. These are visual cues, not enforcement mechanisms — but they make review debt visible. [[claude-ctrl]] takes this further with hooks and SQLite enforcement, and [[Pre-Commit Lint Checks]] makes it deterministic. Heidenstedt's approach is lower-tech but immediately actionable.

### Debug systems built for AI consumption

Principle 3 is the most underrated idea in the piece. Rather than handing the AI raw logs or CLI access, build debug abstractions that surface summarized, structured information. This is a form of [[Harness Engineering]] — designing the feedforward channel so the AI gets actionable information without drowning in noise.

### Complexity as economic cost

Principle 10 is the piece's most distinctive contribution. It's not "simplicity is elegant" — it's "every unnecessary line costs you tokens and success probability on every future AI task." This gives simplicity a hard economic justification that traditional software engineering arguments lack. It connects to [[Simplicity in the Age of AI-Assisted]] but with a different angle: Heidenstedt cares about ongoing AI economics, not one-time rebuild cost.

## Critical Analysis

This is one of the best practitioner guides in the wiki for bridging the gap between "AI is amazing" and "AI will destroy your codebase." Heidenstedt writes from experience, not theory — the specifics (restart servers between property tests, tag risk levels in comments) feel earned.

The piece's strength is also its limitation: it assumes a solo or small-team context where one person can hold the architecture in their head. At [[Minions — Stripe's One-Shot Coding Agents]] scale (1,000+ unattended PRs/week), the "human decides everything" model breaks down. The twelve principles scale to the point where human attention is the bottleneck, then you need the systems-level approaches in [[Scaling LLMs to Larger Codebases]] and [[Guardrails and Feedback Loops]].

The "debug systems for AI" idea (Principle 3) deserves more development than Heidenstedt gives it. It's a genuinely novel concept — designing observability for non-human consumers — that could be its own article. The one-paragraph treatment undersells a significant architectural insight.

Compared to [[Addy Osmani's Workflow]] (which covers similar ground at a higher level), Heidenstedt's piece is more concrete and more paranoid — and the paranoia is justified. The AI-will-cheat warning (Principle 5) is the most important thing a new AI-assisted developer can internalize, and she states it without euphemism.

The security annotation system (Principle 9) is clever but fragile. Comment-based tagging works until someone forgets to update it — which is the exact failure mode [[claude-ctrl]] addresses with enforcement-by-hooks. Heidenstedt's approach is a good first step; production systems need the deterministic version.

---
*Sources: [[summary/how-to-effectively-write-quality-code-with-ai]]*
*Last updated: 2026-05-15*
