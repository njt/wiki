# Parse, Don't Validate

Alexis King's 2019 essay crystallises type-driven design into a single, portable slogan: parsing preserves information in the type system, validation throws it away. The argument is Haskell-native but the insight translates to any language with a type system worth using — Rust's `Result`, TypeScript's discriminated unions, even C#'s `DateTimeOffset` over `DateTime`. When you validate, you check and move on. When you parse, you return a *refinement* of the input type that the compiler then enforces everywhere downstream. It's the difference between a runtime guard and a compile-time proof.

---

## Key Quotes

> "The difference between validation and parsing lies almost entirely in how information is preserved."

This is the essay's thesis in a sentence. `validateNonEmpty :: [a] -> IO ()` learns that a list is non-empty and immediately discards that knowledge. `parseNonEmpty :: [a] -> IO (NonEmpty a)` returns a value whose *type* carries the proof. Any caller that receives a `NonEmpty a` can call `head` on it without a `Maybe` wrapper — the compiler knows the empty case is impossible. The practical difference is that the second version eliminates a whole class of redundant checks and the bugs they enable.

> "Shotgun parsing is a programming antipattern whereby parsing and input-validating code is mixed with and spread across processing code — throwing a cloud of checks at the input, and hoping, without any systematic justification, that one or another would catch all the 'bad' cases."

King borrows this from the LangSec literature (*The Seven Turrets of Babel*, 2016). The diagnostic is precise: validation that returns `()` can always be omitted without the compiler noticing. Shotgun parsing isn't about missing checks — it's about checks that can't be *proved* complete. The consequence is programs that can't reject invalid input before acting on it, leaving program state unpredictable after a late-discovered error. This matters for security: a program that acts on partially-validated input before discovering invalidity has already committed state it may not be able to roll back.

> "Write functions on the data representation you *wish* you had, not the data representation you are given."

This is the design heuristic that makes the slogan actionable. Start from the ideal type signature, then work backward to bridge the gap between what you have and what you need. King's example: if you realise a function needs a `Map` (no duplicate keys) but callers have `[(k, v)]`, change the function signature to require `Map`, follow the type errors up the call chain, and insert the conversion exactly once at the boundary where duplicates are resolved. The typechecker becomes your refactoring assistant, flagging every site that needs updating.

> "Avoid the temptation to just stick a `Bool` in a record somewhere because it's needed by the function you're currently writing."

Booleans in records are the canary for type-driven design debt. A `Bool` that means "is this list non-empty?" should be a `NonEmpty` type. A `Bool` that means "is this config validated?" means the validation wasn't done at the boundary. Every bare `Bool` is a missed opportunity to encode an invariant in a type.

> "Use an abstract `newtype` with a smart constructor to 'fake' a parser from a validator."

The practical escape hatch. When making an illegal state truly unrepresentable is impractical (e.g., constraining an integer to a range), encapsulate the check behind a smart constructor and hide the raw constructor. It's not compile-time proof, but it's a single, auditable choke point rather than checks scattered across the codebase. This pattern is pervasive in Rust (`NonZeroU32`), Haskell (`Positive` from refinement libraries), and increasingly TypeScript (branded types).

## Key Themes

#type-driven-design #parsing #type-safety #design-pattern #functional-programming #langsec

### The Boundary Principle

All parsing should happen at the boundary of the system — the JSON deserialization layer, the CLI argument parser, the HTTP request handler — before *any* business logic touches the data. Once inside the system, types carry the proofs and no further checks are needed. This is the same architecture as [[Cloudflare Security Audit Skill]]'s recon-before-action pattern and [[Designing a Passively Safe API]]'s atomic phases: do the dangerous work once, at the edge, so the core never needs to distrust its inputs.

### Information-Preserving vs Information-Discarding

The essay's most portable insight: functions that return `()` after a check are a code smell. If the purpose of an effectful function is to raise an error on bad input, it should return a refined type instead. This turns a side-effecting validator into a pure(ish) parser — the effect (failure) is separable from the information gain (the refined type). `checkNoDuplicateKeys :: [(k,v)] -> m ()` becomes `parseNoDuplicateKeys :: [(k,v)] -> m (Map k v)`. The `m ()` version can be forgotten; the `m (Map k v)` version *must* be used because the program needs the `Map` to proceed.

### Types as Proof Objects

`NonEmpty a` is not just a data structure — it's a *proof* that the list contains at least one element. The proof lives in the type system and is checked at compile time. This is the functional-programming analogue of [[Correct by Construction]]'s anchor/attribute/link framework: both argue that correctness should be a structural property of the data rather than a check performed after the fact. The difference is that King's approach uses the language's type system as the enforcement mechanism, while Correct by Construction uses normalized schemas. The two are complementary: types prevent logic errors, schemas prevent data quality errors.

### Shotgun Parsing as a Security Pattern

The LangSec connection elevates the essay beyond a style guide. [[Security and Sandboxing]] and [[VulnHunter]] both emphasize that validation spread across a codebase is a vulnerability surface — you can't audit what you can't locate. King's contribution is showing that the fix isn't better validation discipline but *parsing* — returning types that make the checks structurally mandatory. A function that returns `NonEmpty a` can't be called without the caller having proof of non-emptiness; a function that calls `validateNonEmpty` and proceeds can't prove it remembered to.

## Critical Analysis

The essay's strength is its narrowness. King doesn't try to explain all of type-driven design — she explains one distinction and shows how much follows from it. The `NonEmpty` example is perfectly chosen: small enough to fit in a paragraph, rich enough to demonstrate the payoff, and real enough that every Haskeller has encountered the `head` partiality problem.

The essay's limitation is its Haskell-centrism. The slogan translates — Rustaceans reach for `NonZeroU32` and `Parse, Don't Validate` is widely cited in the Rust community — but the *feel* of the refactoring process King describes depends on a type system that makes breaking changes cheap to propagate. In TypeScript, changing a function signature from `Array<T>` to a branded `NonEmptyArray<T>` won't light up every call site the way GHC does; you need `strict: true` and even then the structural type system lets more slide. In Go, the absence of sum types means you're choosing between the `Maybe` approach (return `T, bool`) and the `NonEmpty` approach (a custom struct), and neither is as ergonomic. In Python or Ruby, the distinction collapses entirely — you're writing validators, not parsers, and hoping test coverage catches the gaps.

The second limitation is more subtle: the essay treats "push parsing to the boundary" as unproblematic, but real-world boundary parsing often requires context-sensitive decisions. King acknowledges this briefly ("plenty of useful parsers are context-sensitive") but doesn't explore the tension. A JSON API that parses `{ type: "cat", ... }` differently from `{ type: "dog", ... }` can't fully parse at the boundary without the business logic that distinguishes cats from dogs. The practical answer — parse in layers, with each layer refining the type — works but costs more engineering than the essay suggests.

That said, the essay has aged remarkably well. Written in 2019 for a Haskell audience, it's become one of the most-cited design principles in the Rust community and increasingly in typed-functional TypeScript circles. The rise of [[Constraint Decay]] research — where coding agents collapse under accumulated structural constraints — makes King's argument more relevant, not less: the solution to constraint decay isn't to abandon constraints but to encode them once, at the boundary, in types that the compiler enforces. An agent that must produce a `NonEmpty a` to call your function can't forget the check; the type system won't let it compile.

The core idea is simple enough to state in three words. The hard part, as King is honest about, is developing the taste to know which refinements are worth encoding and which aren't. Not every invariant belongs in the type system. But the ones that do pay rent forever.

---

*Sources: [[raw/parse-don-t-validate]], [[summary/parse-don-t-validate]]*
*Last updated: 2026-08-08*
