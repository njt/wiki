# The Wrong Abstraction

Sandi Metz's compact, career-altering essay on why duplicated code is cheaper than the wrong abstraction — and the step-by-step pattern by which a well-intentioned abstraction decays into a condition-laden mess that nobody can understand but everyone feels obligated to preserve. The essay distills a throwaway line from her RailsConf 2014 talk into a standalone argument that has become one of the most-cited pieces of software design advice of the past decade.

---

## Key Quotes

> "duplication is far cheaper than the wrong abstraction"

The essay's thesis, stated as a flat assertion. The word "far" does the work here — Metz isn't saying duplication is good, she's saying the *relative* cost of the wrong abstraction is so much higher that duplication is the safer default. This inverts decades of DRY-first teaching. The claim is counterintuitive enough that it provoked strong reactions when she first made it, ranging from "you've lost your mind" to "this, a million times this."

> "Existing code exerts a powerful influence. Its very presence argues that it is both correct and necessary."

The mechanism by which wrong abstractions survive. Code doesn't need to be good to be persuasive — it just needs to *exist*. The mere fact that someone invested effort in creating it creates a presumption of correctness. This is a psychological observation, not a technical one, and it's the essay's deepest insight: the forces that preserve bad abstractions are in our heads, not in our editors.

> "the sad truth is that the more complicated and incomprehensible the code, i.e. the deeper the investment in creating it, the more we feel pressure to retain it (the 'sunk cost fallacy')"

The cruel irony at the heart of the pattern. The worse the abstraction, the stronger the gravitational pull to keep it. Complexity masquerades as sophistication: "Goodness, that's so confusing, it must have taken *ages* to get right." The sunk cost fallacy isn't just an economic concept Metz borrows — it's the name for a specific failure mode in software design that her seven-step pattern makes legible.

> "the fastest way forward is back"

The essay's prescription, and its most counterintuitive advice. When you're stuck in a wrong abstraction, adding more parameters and conditionals is the slow path. Undoing the abstraction — re-introducing duplication, inlining the code back into callers, trimming each to what it actually needs — is faster. Metz frames this as advance, not retreat: "This is not retreat, it's advance in a better direction." The reframe matters because it gives engineers permission to do something that feels like destruction.

> "Once you inlined the code, the path forward became obvious, and adding new features become faster and easier."

The payoff. The wrong abstraction doesn't just make the current feature hard — it *obscures* what the right abstraction would be. By inlining, you let the actual usage patterns surface. Each caller reveals what it truly needs, and from that concrete evidence, better abstractions emerge.

---

## The Seven-Step Pattern

Metz's pattern is the essay's most cited contribution. It's worth reproducing in full because it's a diagnostic tool: if you recognize yourself in step 6, you're in the wrong abstraction.

1. Programmer A sees duplication.
2. Programmer A extracts duplication and gives it a name.
3. Programmer A replaces the duplication with the new abstraction.
4. Time passes.
5. A new requirement appears for which the abstraction is *almost* perfect.
6. Programmer B adds a parameter and conditional logic.
7. Loop (more requirements → more parameters → more conditionals) until incomprehensible.

The critical step is #6. The decision to add a parameter rather than question the abstraction is where the decay begins. Each subsequent parameter makes questioning the abstraction *harder* (more sunk cost), accelerating the trap.

---

## Key Themes

- **#concept Wrong abstraction cost** — the core idea: duplicated code's cost is linear and bounded; wrong abstraction's cost is compounding and unbounded
- **#pattern Inline-then-re-extract** — the prescribed escape: undo the abstraction, let callers reveal their true needs, then abstract from evidence
- **#concept Sunk cost fallacy in code** — complexity creates preservation pressure; the worse the code, the harder it is to delete
- **#pattern Abstraction decay cycle** — the seven-step pattern from clean extraction to condition-laden mess
- **#person Sandi Metz** — author of [[99 Bottles of OOP]], one of the clearest voices in practical OO design

---

## Critical Analysis

**The essay's brevity is its strength.** At ~800 words of argument (the rest is a book announcement), Metz doesn't over-elaborate. The seven-step pattern is specific enough to recognize in your own codebase but general enough to apply across languages and paradigms. The sunk cost fallacy connection is stated but not belabored. The result is an essay you remember and quote, not one you skim and forget.

**What's left unsaid: when *is* abstraction right?** Metz's advice is "prefer duplication over the wrong abstraction" — not "never abstract." The essay is a corrective aimed at overcooked DRY instinct, not a complete theory of when to abstract. The implicit rule is: abstract when you have evidence from multiple concrete cases that the abstraction captures a genuine commonality. In practice, that means waiting through several instances of duplication before extracting — letting the pattern prove itself. This is the same waiting discipline that Kent Beck's [[The Cost YAGNI Was Never About]] argues for: waiting is holding an asset, not laziness.

**The relationship to [[The Economic Benefit of Refactoring]] is a productive tension.** Fowler measures the token-cost savings of *good* refactoring — cleaner decomposition means agents read 83% fewer tokens per change. Metz warns about *bad* abstraction — the condition-laden mess that makes every change harder. The synthesis: refactoring toward better decomposition is valuable, but the direction matters. Good abstractions reduce cognitive and token load; wrong abstractions increase both. Fowler's experiment shows what happens when you refactor toward *better* structure. Metz's essay diagnoses what happens when you preserve *wrong* structure. The practical takeaway: refactor, but be willing to inline first if you're not sure the abstraction is right.

**The sunk cost framing connects to [[Engineering for Bounded Cognition]].** Williams argues that working memory (~4 chunks) is the binding constraint on software engineering. Sunk cost fallacy is one of the failure modes of that constraint: we can't hold the abstraction *and* the new requirement *and* the conditional logic in our heads simultaneously, so we default to preserving the abstraction — it's already there, already "understood," already invested in. The cognitive load of questioning the abstraction (what would inlining look like? what would the callers actually need?) exceeds our working memory, so we take the path that adds one more parameter instead. Metz's advice to "go back" is, in Williams's terms, a cognitive offload: inlining moves the complexity out of your head and onto the page where you can actually see it.

**The essay predates the AI coding era but its lessons compound.** Coding agents are natural wrong-abstraction amplifiers. They see duplication, eagerly extract it (step 2), and produce abstractions at a rate no human would. But they also lack the taste to know when an abstraction has gone wrong — they'll happily add the sixth parameter and seventh conditional without questioning the structure. The Metz pattern, applied to agentic development, suggests that agent-generated abstractions need *more* scrutiny, not less, precisely because they're produced so cheaply. Cheap generation doesn't make wrong abstractions cheaper — it makes them faster to accumulate.

**Where the rule doesn't apply: duplication you can delete outright.** Kent Beck's [[Composable Tests]] argues for removing redundant assertions from a test suite — which looks like exactly the DRY reflex Metz warns against, but isn't, because Beck's remedy is *deletion*, not extraction. No name is created, no shared helper, no base class; nothing gains a claim on the system. That sharpens Metz's boundary usefully: her rule is that duplication is cheaper than the *wrong abstraction*, so it says nothing about duplication that can be removed without building an abstraction at all. The dangerous misreading in the other direction is equally instructive — a developer who responds to redundant tests by extracting a shared `assertItWorked()` helper has walked straight into step 2 of Metz's pattern.

**The strongest unstated implication: naming is the trap.** Step 2 says "gives it a name." The name creates the thing. Before the name, it's just similar-looking code. After the name, it's an *abstraction* — a concept with a right to exist. The name is what makes Programmer B feel honor-bound to preserve it. Metz doesn't say this explicitly, but the pattern implies that the act of naming is the point of no return. This connects to [[99 Bottles of OOP]] where Metz treats naming as the central act of design — a named thing has a claim on the system that unnamed duplication does not.

---

*Sources: [[raw/the-wrong-abstraction]], [[summary/the-wrong-abstraction]]*
*Last updated: 2026-08-08*
