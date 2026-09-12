# Push Ifs Up And Fors Down

Two related code-structure heuristics from Alex Kladov (matklad): centralize control flow at the highest level, and batch data operations at the lowest level. Together they form a practical rule of thumb for writing code that is both easier to reason about and faster to execute.

---

## The Heuristics

### Push Ifs Up

> If there's an `if` condition inside a function, consider if it could be moved to the caller instead.

This surfaces most naturally with preconditions. A function that checks a precondition internally and silently does nothing when it fails is hiding decision-making. Moving the check to the caller — enforced via types or an assert — makes the contract explicit. Done systematically, this can reduce the total number of checks across the system.

The deeper argument is cognitive: control flow is complicated, and `if` statements are a source of bugs. By pushing them up, you centralize branching logic in a single function where it fits on one screen. From that vantage, redundancies and dead conditions become visible — something much harder to spot when the same logic is scattered across multiple subroutines.

> With all the flow in one place it often is possible to notice redundancies and dead conditions.

A related technique Kladov calls the **"dissolving enum" refactor**: when multiple functions branch on the same enum variant, pulling the branch up collapses triplicated conditions into a single dispatch. The data structure that reified the condition disappears.

### Push Fors Down

> Few things are few, many things are many.

From the data-oriented design tradition: programs usually operate on collections of objects, and the hot path involves handling many entities — that volume is what makes the path hot in the first place. So make batch operations the default, and treat scalar operations as a degenerate case of `batch_size = 1`.

The primary motivation is performance:
- Amortize setup costs across the batch
- Choose processing order freely (not per-entity)
- Unlock vectorization and struct-of-array layouts
- The extreme case: FFT-based polynomial multiplication, where evaluating a polynomial at many points simultaneously is faster than individual evaluations

But Kladov notes expressiveness benefits too: jQuery succeeded by operating on collections of elements, and the language of abstract vector spaces is often a better tool for thought than coordinate-wise equations.

## Composition

The two rules compose neatly:

> The `GOOD` version avoids repeatedly re-evaluating `condition`, removes a branch from the hot loop, and potentially unlocks vectorization.

This pattern scales from micro (a single loop) to macro (system architecture). Kladov cites TigerBeetle as an example: the data plane operates on batches of objects to amortize the cost of decision-making in the control plane — pushing `for`s down and `if`s up at the architecture level.

## Analysis

These aren't novel ideas — they're restatements of principles found in data-oriented design, refactoring literature, and performance engineering. Their value is in the compression: two memorable, teachable rules that cover a lot of ground.

**What's strong:** The "viral" property of pushing `if`s up is real and underappreciated. When a precondition check moves upward, it often eliminates sibling checks in sibling callers — a compounding benefit that isn't obvious from a single example. The composition observation (push the `if` above the `for`) is also undervalued in practice; developers often reflexively put the check inside the loop body where it wastes cycles on every iteration.

**What's missing:** The post doesn't address the tension between the two rules. Pushing `for`s down into batch APIs can make the `if`-pushing harder — a batch function with internal conditionals (e.g., filtering) resists both patterns simultaneously. The escape hatch is to separate filtering (`if`-pushing territory) from processing (`for`-pushing territory), but Kladov doesn't make this explicit.

**The TigerBeetle connection:** This isn't just a blog post — it's a window into the architectural philosophy behind one of the most performance-sensitive databases in production. The "batches amortize control-plane costs" insight is a genuine contribution that goes beyond textbook data-oriented design.

## Related

- [[The Economic Benefit of Refactoring]] — empirical measurement of how structural decomposition (related to pushing `if`s up) reduces cognitive and token load
- [[99 Bottles of OOP]] — Sandi Metz's structured approach to refactoring; the Flocking Rules as a systematic version of the "push ifs up" intuition
- [[Performance Optimization Loop]] — the measurement-driven discipline that determines *whether* pushing `for`s down actually helps in a given hot path
- [[Code Cleanliness and Coding Agents]] — how code structure (including centralized control flow) affects agent effectiveness and token consumption
- [[Software Engineering Craft]] — hub page for enduring software design principles

---
*Sources: [[raw/push-ifs-up-and-fors-down-html]], [[summary/push-ifs-up-and-fors-down-html]]*
*Last updated: 2026-08-08*
