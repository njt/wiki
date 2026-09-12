# Type Safety Back and Forth

Matt Parsons' framework for type-driven design: a function can either push failure responsibility forward (return `Maybe`, make callers handle it) or push it backward (demand stricter arguments that can't fail). Both are valid, but the backward push is the rarer and more powerful technique — and it naturally propagates until all uncertainty lives at the system's edges.

---

## Key Quotes

> "I like to think of this as pushing the responsibility for failure forward. I'm telling the caller of the code that they can provide whatever `Int`s they want, but that some condition might cause them to fail."

Parsons names the familiar pattern: `safeDivide :: Int -> Int -> Maybe Int`. This is the technique every Haskell tutorial teaches — wrap the result in `Maybe` or `Either` and let the caller figure out what to do with failure. It's "one-size-fits-all" and easy to show in a 35-line blog post. The trade is that every caller now carries the cognitive load of handling a `Nothing` case, and that load compounds as functions compose.

> "No! You must provide a `NonZero Int`. I refuse to work with just any `Int`, because then I might fail, and that's annoying."

The backward push speaks with a different voice. Instead of `safeDivide :: String -> String -> Maybe Int` (which can fail parsing, dividing by zero, all returned as `Nothing`), the backward version is `safeDivide :: Int -> NonZero Int -> Int` — total, pure, cannot fail. The function has pushed the proof obligation onto the caller: *you* prove the divisor is non-zero, and I'll give you a guaranteed result. The `PatternSynonyms` extension lets the library expose `NonZero` for pattern matching without exposing its unsafe constructor, making the type a one-way door: values come out safely, but new values must go through `nonZero :: a -> Maybe (NonZero a)`.

> "All of the `Maybe`s relating to the empty list were purged, and all of the code was pure and free."

The article's most valuable section describes a real production refactoring. An `Order` type originally stored `items :: [Item]`, which meant nearly every function that touched an order had to handle the empty-list case with `Maybe`. But an order without items is a business impossibility — the type was *too permissive*. Changing to `items :: NonEmpty Item` eliminated all those `Maybe` values at a stroke. The empty-list failure case was pushed all the way to exactly two edge sites: JSON decoding (return HTTP 400 to API clients with bad data) and database reads (enforced via `INNER JOIN` + foreign keys).

> "By restricting our past, we gain freedom in the future."

The thesis compressed to a sentence. Parsons connects this to the `justified-containers` library, which uses the type system to prove that a key exists in a `Map` — lookups become total, `Maybe`-free operations that survive `map` transformations and can even prove relationships across maps. When you design with type safety, you aim to be as strict as possible about inputs. A `ByteString -> IO ByteString` function can accept anything and break anywhere; a function that takes a `NonEmpty (Map UserId Order)` has dramatically fewer failure modes, and that simplification compounds through every layer that calls it.

## Key Themes

#type-safety #haskell #design #pattern #api-design #parsing

- **Forward vs. backward failure propagation.** Forward = `Maybe`/`Either` on the output; backward = restrictive types on the input. Both are type-safe, but they distribute cognitive load differently: forward spreads it across all callers, backward concentrates it at system boundaries.

- **Making illegal states unrepresentable.** The backward push is the operationalization of Yaron Minsky's famous principle. By using `NonZero Int` instead of `Int` with a runtime check, `NonEmpty a` instead of `[a]` with a `Maybe` guard, you encode constraints in types so the compiler enforces them.

- **Types that match business reality.** The `Order` refactoring is the clearest illustration: the business rule "every order has at least one item" should be reflected in the type. When the type is more permissive than reality, you pay for that mismatch in unnecessary `Maybe` handling everywhere. Tightening the type to match the domain eliminates entire categories of bugs.

- **Edge concentration as architecture.** Push type safety backward far enough, and all uncertainty concentrates at the system's two natural edges: the API boundary (validate incoming data, reject bad requests) and the database layer (enforce constraints at the schema level). Everything between is total, pure, and trivially composable. This is the type-level version of functional core / imperative shell.

- **PatternSynonyms as encapsulation.** Haskell's `PatternSynonyms` lets library authors export a pattern for matching without exporting the constructor for building. This creates types that are one-way: you can inspect values safely, but constructing new ones requires going through the smart constructor (`nonZero :: a -> Maybe (NonZero a)`), which forces callers to handle the validation case.

## Critical Analysis

This is a crisp, honest blog post that does something rarer than it seems: it names a pattern that experienced Haskell programmers feel intuitively but rarely articulate. The forward/backward framing is an excellent teaching tool because it's symmetric and visual — you can point at the type signature and say "the failure responsibility moves this direction."

The limitation is that Parsons only glances at the trade-offs. The backward push requires writing new wrapper types and smart constructors for every constraint, which has a real syntactic and maintenance cost. In a language like Haskell, `newtype` + `PatternSynonyms` keeps the runtime cost at zero (newtypes are erased), but the design cost is non-trivial: you now have `NonZero`, `NonEmpty`, `NonBlank`, `Positive`, `Validated`, and a hundred other domain-specific wrappers, each with its own module and smart constructor. The Haskell ecosystem has mostly settled on "use `Maybe` for simple cases, reach for type-level proofs when the payoff justifies the infrastructure" — but Parsons elides this ergonomic reality.

The production `Order` refactoring is the strongest section, but it's brief. I'd want to hear about the migration: how many functions broke when `[Item]` became `NonEmpty Item`? What errors did the compiler surface that manual review would have missed? How did the PR review go? These details would make the case more concrete for skeptics who see type-level proofs as academic exercises.

The connection to Alexis King's "Parse, Don't Validate" (2019) is notable by its absence — this post from 2017 is arguably the precursor, articulating the same idea from the other direction. King's article is about *when* you narrow types (at the boundary), while Parsons is about *whether* you narrow types at all. Together they form a complete picture of type-driven design that neither captures alone.

---

*Sources: [[raw/type-safety-back-and-forth-html]], [[summary/type-safety-back-and-forth-html]]*
*Last updated: 2026-08-08*
