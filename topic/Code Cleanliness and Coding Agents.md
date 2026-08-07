# Code Cleanliness and Coding Agents

The first controlled study that varies the *codebase* while holding the agent fixed — and finds that cleaner code doesn't change whether agents succeed, but does change how efficiently they work: 7–8% fewer tokens consumed and a striking 34% reduction in file revisitations.

---

Trivedi and Schmitt (SonarSource) constructed six minimal-pair repositories — behavioral twins that differ only on static-analysis rule violations and cognitive complexity — and ran 660 trials with Claude Code (Sonnet 4.6) across 33 tasks. This inverts the standard evaluation setup: instead of varying the agent against a fixed benchmark, they vary the code against a fixed agent. It's an obvious-in-retrospect design that nobody had executed before.

The headline is that pass rates stay flat (0.913 clean vs. 0.921 messy — a rounding error). The clean side doesn't make agents *better*, it makes them *cheaper*.

## Key Findings

**File revisitation is the clearest signal.** Across every repository, agents return to files they've already edited ~34% less often on cleaner code. The interpretation: on messy code, agents make an edit, leave, then come back to double-check because they're not confident they understood the context. On clean code, they read wider on the first pass (+3.2% files read) and move on.

**Multi-module tasks are where cleanliness pays most.** When tasks span two or more modules, input tokens drop 10.7% and revisitations drop 50.8% on the clean side. Cognitive-hotspot tasks (dense single-method complexity) show essentially no token difference — the cleanup pipeline redistributes complexity across more methods rather than eliminating it, which can actually *increase* surface area for the agent.

**Variance dominates everything.** Within the same (task, side) combination, the most expensive trial is typically 2.5× the cheapest. Per-task input-token deltas span -47% to +44%. Cleanliness is real but it's a weak signal swimming in a sea of agent non-determinism.

## The Two Case Studies That Explain Everything

> "On the cleaner repo, agents use 35% fewer input tokens, open 25% fewer files, edit 31% fewer lines, and take 32% fewer turns."

**BCEL wins clean.** The messy variant had opcode dispatch in parallel god methods with hundred-line switch statements. The cleanup pipeline replaced them with thin dispatchers delegating to named helpers. Agents could grep for the exact method they needed and edit surgically.

> "The file is roughly the same size on both variants, but the cleaner side spreads surrounding work across more methods. This shows up as an 8% *increase* in input tokens on the cleaner variant."

**Genie loses clean.** The cleanup pipeline extracted helpers around the focal logic but kept the original logic in place. The agent paid for the added surface area without getting simpler access to the thing it actually needed to change.

The distinction: *thin dispatchers over god methods* help agents; *more methods without better decomposition* hurt them. Cleanliness isn't a single variable — the *kind* of cleanliness matters.

## Critical Analysis

**The study validates what senior engineers already know, but quantifies it.** Nobody who's maintained a codebase will be surprised that cleaner code is easier to navigate. But having a number (7–8% tokens, 34% revisitations) changes the conversation from intuition to economics. At 4M tokens per SWE-bench task, an 8% savings is 320K tokens — multiply by a team running agents daily and it's real money.

**The single-agent limitation is the biggest gap.** All 660 trials used Claude Sonnet 4.6. The authors tried Haiku 4.5 and found it too noisy. Would GPT-5 or Gemini show the same pattern? Would a weaker model benefit *more* from cleanliness (because it needs all the help it can get) or *less* (because it's random anyway)? This paper opens a question it can't answer.

**The lack of compounding measurement is the elephant in the room.** 7–8% per task is modest. But if agents are editing the same codebase repeatedly, do cleanliness effects compound — does cleaner code *stay* cleaner under agent work, producing an accelerating advantage? Or does agent work degrade cleanliness regardless of starting point? The authors flag this as future work; it's the question that determines whether this finding is a curiosity or a practice-changing insight.

**The "cleaner" and "messier" labels are relative, not absolute.** Three of the six minimal pairs are private (to prevent memorization), which means we can't inspect them. The "messier" side of a pair might still be cleaner than the average production codebase. The effect size might be larger in the wild.

**The study accidentally makes the case for pre-commit linting on steroids.** If static-analysis violations cost tokens and revisitations, and agents can auto-fix most violations, the ROI on running SonarQube (or any linter) before an agent touches a file is straightforward to calculate. This paper gives you the numbers to justify that CI step.

## Connections

This paper sits at the intersection of [[Guardrails and Feedback Loops]] (linters as agent infrastructure, not just human guardrails) and [[Agent Coding Workflow]] (the economics of agent-driven development). The finding that comment volume isn't the driver nods toward [[Engineering for Bounded Cognition]] — structure matters more than explanation.

The constraint decay paper by Dente et al. finds that "convention-heavy frameworks are a trap for agents" — this paper's Genie case study echoes that: adding structure without improving decomposition is worse than leaving things alone. [[Constraint Decay]] is the closest cousin in the existing wiki.

The token-consumption findings connect to the broader theme of agent economics: [[Writing Code vs. Shipping Code]] (AI gains attenuate at release level), [[How Hightouch Built Their Long-Running Agent Harness]] (context management is the real cost), and [[Uber — Agentic Engineering Shift]] (6× cost explosion since 2024). Cleanliness is one lever among many for controlling that cost.

Martin Fowler's refactoring experiment ([[The Economic Benefit of Refactoring]]) takes this finding to its logical extreme: 15 aggressive refactoring steps on a 17K-line agent-generated monolith reduced input tokens by 83% — an order of magnitude beyond SonarSource's 7–8%. The two studies together suggest a dose-response relationship where surface cleanup yields surface savings and deep semantic decomposition yields deep savings. The key mechanism is the same in both: the agent reads fewer files, not because there's less total code (lines stayed roughly constant in both), but because decomposition lets it confidently identify the right subset.

The minimal-pair methodology itself is worth noting alongside [[Simon Willison — Engineering Practices That Make Coding Agents Work]] — both are attempts to move from "I feel like this works" to "here's evidence it works."

---

*Sources: [[raw/code-cleanliness-coding-agents]]*
*Last updated: 2026-07-08*
