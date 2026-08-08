# Libraries over Frameworks

Tomas Petricek's 2015 manifesto on why libraries beat frameworks: frameworks own the control flow and resist composition, exploration, and testing; libraries are called by your code and compose freely. The article is a functional programmer's design guide — all F# examples — but the principles are language-agnostic and have become only more relevant as LLM coding agents struggle with the implicit magic that frameworks rely on.

---

## Key Quotes

> "When using a framework, the framework is in charge of running the system. It defines some extensibility points (interfaces) where you need to put your implementation. When using a library, you are in charge of running the system."

The one-paragraph distinction that the whole article builds on. A framework is the Hollywood Principle ("don't call us, we'll call you") made architectural. A library is a tool you pick up and use. The rest of the article is about why this distinction matters and how to stay on the library side of it.

> "Frameworks do not compose. When you have two frameworks, they both force you to fill in a specific hole. But there is usually no way to fit one framework inside another."

This is the article's strongest claim and the one that has aged best. In 2015, the composition problem was a developer-experience annoyance. In 2026, with coding agents generating code across stacks, it's a structural blocker. An agent that encounters two frameworks with competing control-flow expectations has no clean resolution — and unlike a human, it won't recognize the conflict as a design problem rather than a coding problem.

> "The difference between libraries and frameworks is pretty much the same as the difference between calling a function and having to provide a function as an argument."

Petricek formalizes the distinction with type signatures. A library is `lib : τ₁ → τ₂` — you create a value, call the function, get a result. A framework is `fwk : (σ₂ → σ₁) → unit` — you provide a function, the framework calls it, and you get nothing back (only side effects). This is why libraries are explorable in a REPL and frameworks aren't: you can experiment with different `τ₁` values and see what comes back, but you can't experiment with `σ₁` values without understanding the framework's internal state.

> "It is fine to have an easy-to-use operation that takes a couple of other functions as a high-level abstraction, but there should be a simple and more explicit alternative!"

The escape hatch principle. Petricek isn't arguing against convenience functions — he's arguing against convenience functions that are the *only* interface. The `startGame` function takes `update` and `draw` callbacks, which looks like a framework, but it's built on top of a lower-level event-based API that gives you full control. You can always drop down one level and write the 14-line async loop yourself. This is the same pattern as [[The GUS Stack — Go, Unix, SQLite]]: the stack provides sensible defaults while leaving the escape hatches visible.

> "With traveling, I can see why some people prefer package holidays, but in software development, there is no excuse for building frameworks!"

The parting shot. Package holidays (frameworks) handle everything but constrain what you can do. Independent travel (libraries) is more work but gives you control. Petricek's claim is that in software, the control always matters — the framework's constraints always eventually become the wrong shape for your problem.

## Key Themes

#library-design #framework #composition #api-design #functional-programming #inversion-of-control #abstraction #pattern

- **The framework/library distinction as control-flow ownership.** A framework owns `main()`; a library is called from your `main()`. Everything downstream — composability, testability, explorability — follows from this single choice. This isn't a spectrum; it's a binary that becomes apparent the moment you try to use two things together.

- **Composition as the killer argument.** Libraries compose; frameworks don't. This is the claim that makes the article more than a matter of taste. Petricek notes the structural similarity to monads — both are easy to get into and hard to get out of, and composing either requires a swap operation (`M₁(M₂ α) → M₂(M₁ α)`) that rarely exists. [[Constraint Decay]] provides the 2026 empirical confirmation: convention-heavy frameworks are a trap for coding agents precisely because their implicit behavior resists composition with other tools.

- **Interactive exploration as a design discipline.** Petricek's most practical advice: design your library so it can be explored from a REPL. Have one obvious entry-point type. Make its methods discoverable through autocomplete. If you can't write a one-line F# Interactive script that calls your library, your API surface is too complex. This is the same instinct behind [[10 Principles for Agent-Native CLIs]]: design for the discoverability of the tool, not just its functionality.

- **Multiple levels of abstraction as an ethical obligation.** The high-level convenience API is fine — but only if there's a lower-level, more explicit alternative one step beneath it. The user should always be able to take the convenient 4-line version, look at its definition, and understand the 14-line version it compiles to. Without the lower level, the convenience is a trap.

- **Callbacks as a spectrum, not a sin.** Petricek is careful not to condemn all higher-order functions. A single, stateless callback (like `List.map`) is fine. Multiple callbacks that share state are the smell. The fix is splitting the function into independent pieces — `validateInput` (a pure function) and `ignoreIOErrors` (a single-callback wrapper) rather than `readAndProcess` (two callbacks with implicit state threading).

- **Events + async as the inversion-of-control antidote.** Rather than requiring users to implement `Update()` and `Draw()` methods on a base class, expose `Update` and `Draw` as events and let users write their own control loop with `async { }` blocks. The user stays in control of initialization, state management, and termination — the framework only signals when work is needed. This is the same pattern behind [[Phoenix LiveView]]'s event-driven architecture and [[Signals — The Push-Pull Algorithm]]'s reactive model.

## Critical Analysis

**The article's argument has aged asymmetrically.** The composition problem has gotten dramatically worse — modern web development stacks often involve three or four frameworks (React, Next.js, Prisma, tRPC) that each want to own part of the control flow, and the result is exactly the non-composition Petricek warned about. The exploration problem has been partially solved by LSP servers and IDE autocomplete, which give framework users some of the discoverability that REPL users had in 2015. But the core asymmetry remains: you can't load a Next.js app into a REPL and call individual route handlers with test inputs.

**Constraint Decay vindicates Petricek from an unexpected direction.** The 2026 finding that convention-heavy frameworks (Django, FastAPI) cause 30pp drops in agent pass rates while explicit frameworks (Express, Flask) hold up better is exactly Petricek's argument, confirmed empirically for a user he never imagined: the LLM coding agent. The framework's magic — the implicit middleware, the dependency injection, the convention-over-configuration — is what makes it powerful for humans and illegible to agents. Petricek argued that frameworks don't compose with *other libraries*; [[Constraint Decay]] shows they also don't compose with *the tool that writes the code*.

**The F# framing is both a strength and a limitation.** Petricek's examples are clear and the type-signature formalization is genuinely useful. But the article assumes a language with algebraic data types, async workflows, and first-class events — features that are absent or awkward in many mainstream languages. The principles translate, but the concrete techniques (AwaitObservable, computation expressions) don't. A Python or JavaScript developer reading this in 2026 will nod along with the diagnosis and then ask "so what do I actually do in my language?" — a question the article doesn't answer.

**"No excuse for building frameworks" is false, and the article's own Mario example shows why.** The final `startGame` function — the high-level convenience API — *is* a framework by Petricek's own definition. It takes callbacks, owns the control flow, and hides the event loop. The difference is that there's a lower-level alternative, which makes the framework a convenience layer rather than a cage. The real principle isn't "never build frameworks" — it's "never build frameworks without an escape hatch." The article says this clearly in the text but buries it under a clickbait title.

**What the article misses: ecosystems matter more than individual library design.** Petricek treats library design as a property of individual libraries — design yours well and the world improves. But composition is an ecosystem property. One well-designed library in a sea of frameworks doesn't help much; the user still has to bridge the gap. The FsLab example (linking Deedle and Math.NET through a thin conversion layer) hints at this but doesn't develop it. The real lesson is that library ecosystems need *integration libraries* — thin adapters that convert between types — and those are undervalued because they're boring.

**The trade tour operator metaphor is good but incomplete.** Petricek compares frameworks to package holidays and libraries to independent travel, then declares there's "no excuse" for frameworks in software. But package holidays exist for a reason: they reduce cognitive load, handle edge cases, and make the 80% case trivially easy. The same is true of frameworks. Rails in 2005 didn't win because it was composable — it won because it made web development accessible to people who would never have built a web app otherwise. The real argument should be: frameworks are fine for the 80% case, but they must have an escape hatch for the 20%. Petricek's own `startGame` example proves he agrees with this; the title just overstates the case.

---

## See Also

- [[Constraint Decay]] — the 2026 empirical confirmation: convention-heavy frameworks cause 30pp drops in agent code generation pass rates
- [[The GUS Stack — Go, Unix, SQLite]] — boring, composable libraries over monolithic frameworks as the agent-optimized stack
- [[Software Engineering Craft]] — the hub page covering API design, simplicity, and the fundamentals that frameworks often obscure
- [[Specifications as the Product]] — if code is disposable, the spec is the durable artifact; frameworks that own the control flow make specs harder to extract

---

*Sources: [[raw/library-frameworks]], [[summary/library-frameworks]]*
*Last updated: 2026-08-08*
