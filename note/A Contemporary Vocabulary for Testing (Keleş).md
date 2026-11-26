# A Contemporary Vocabulary for Testing (Keleş)

Alperen Keleş tears down the standard testing taxonomy — unit/integration/E2E, load/penetration — as undefined "vibes" and rebuilds it as a three-axis design space: the **scope** of the test unit (function to system, stateful or not), the **hypothesis** the test falsifies (correctness, performance, security, resilience), and the **oracle** that judges failure — with oracles cleanly separated from **methods** of generating inputs. His real agenda is promoting random testing, but the decomposition itself is the durable contribution.

---

## The argument

The standard vocabulary fails because "unit" is undefined, "integration" integrates arbitrarily scoped modules, and "E2E" hides the actual tested behavior and method. Keleş's replacement:

1. **Scope**: two orthogonal axes — language construct (function/class/module/system) and statefulness. Stateless functions take arbitrary structured inputs; systems take serialized, validated interactions, which is a natural discriminator that shrinks the input space. State can be lifted into the test's input-output pair, unless realistic states are too complex to construct — then you treat the system as a simulation.
2. **Purpose/hypothesis**: every test falsifies a hypothesis in some domain — functional correctness, timing, security (timing side channels), accessibility, resilience (chaos testing). Each domain needs its own oracle.
3. **Oracles**: human judgement ("tinker testing" — the unticketed root of all testing), examples (`add(3,5) == 8`, snapshot/golden/expect testing), inline assertions (DbC, Tiger Style's production assertions), and properties (global invariants, metamorphic relations, roundtrip, differential/compatibility, idempotency, LTL liveness).
4. **Methods**: static analysis (the type system *is* a test; refinement types statically check the same properties), enumeration, and randomness — generation, mutation, coverage-guided fuzzing, property-based testing, model-based simulation, fault injection and Deterministic Simulation Testing.

The key move is the oracle/method separation. Tinker and example testing fuse the two; properties and inline assertions do not, which is why a property can be checked by fuzzing, enumeration, or a model checker alike.

## Key quotes

> "There is no proper classification in the wild, it all seems to be based on 'vibes'."

The diagnosis lands. Anyone who has argued about whether something "counts as a unit test" is arguing about a word that was never defined — and Keleş is right that the gap matters because vocabulary should be actionable: you should be able to *upgrade or downgrade* a test as resources change.

> "A traditional software test is ultimately a codification of tinker tests."

The best observation in the piece. The loop of poking at your own artifact and squinting at the result is the primal testing act; everything formal downstream is frozen tinkering. It quietly dignifies the way people actually work, including the non-TDD workflow he admits to.

> "The input type of the function includes many potential inputs that the function will never consume."

A precise statement of why function testing feels like fighting the language — and why DbC, refinement types, and domain modelling are testing tools as much as language-design tools. The hashmap example (every lookup fallible to the type system, none fallible in reality) is exactly the right illustration.

> "Compatibility... has found rendering bugs in browsers, compiler bugs in GCC, optimization bugs in SQLite."

Differential testing as the strongest property class, with the receipts. Also the cleanest bridge to distributed systems: "your very complex distributed system should provide the same observable behavior as its simpler, single-node version."

## Themes

#concept #pattern #tool

## Take

This is a genuinely useful essay — not just taxonomy-suffixing, because the oracle/method split actually does explanatory work: it explains why snapshot tests, fuzzers, model checkers, and type systems belong in one conversation, and why "unit vs integration" was never the interesting question. The weaknesses are the ones Keleş flags himself: the scope axis is fuzzy at the system end (entry function of a large system ≈ the system), and the whole piece is a polemic for random testing wearing a neutral-taxonomy costume — DST, fuzzing, and property-based testing get rich treatment while, say, contract-coverage of UI testing gets one line. Still, as a map of the testing design space it is one of the more coherent things written on the subject, and it dovetails with the agent era: agents need oracles even more than humans do, and "the oracle is human judgement" is exactly what stops scaling first.

## Related pages

- [[How SQLite Tests Software]] — SQLite is Keleş's own headline example of what a serious test infrastructure looks like; this essay supplies the vocabulary (scope/hypothesis/oracle/method) for describing what those tests actually are.
- [[Property-Based Testing]] — the essay subsumes PBT as properties-as-oracles plus randomness-as-method, and extends it with the metamorphic, differential, and idempotency property catalogue.
- [[Mutation Testing]] — a method whose oracle is "the test suite itself must catch injected faults," a point in Keleş's space that his taxonomy doesn't name but cleanly accommodates.
- [[Ways of Checking]] — both are attempts to enumerate the whole family of verification techniques; Keleş's oracle/method split gives that page a sharper axis to organize itself by.

---
*Sources: [[raw/vocab-for-testing]], [[summary/vocab-for-testing]]*
*Last updated: 2026-11-26*
