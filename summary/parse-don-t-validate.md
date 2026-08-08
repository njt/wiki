---
url: https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/
title: "Parse, Don't Validate"
author: Alexis King
date_fetched: 2026-08-08
date_published: 2019-11-05
---

# Parse, Don't Validate

Alexis King distills the essence of type-driven design into a three-word slogan: **parse, don't validate**. The essay argues that the difference between parsing and validation lies in how information is preserved: a validator checks input and throws away what it learned (returning `()`), while a parser checks input and returns a *refinement* of the input type that preserves the gained knowledge in the type system.

The core example traces the evolution of a `head :: [a] -> a` function through two approaches. The first weakens the return type to `Maybe a`, pushing the burden of handling empty-list cases onto every caller — even callers that already verified the list is non-empty. The second, and preferred, approach *strengthens* the argument type to `NonEmpty a`, which eliminates the empty case entirely. Once the check is done at the boundary, it never needs to be checked again.

King connects this to **shotgun parsing**, a term from language-theoretic security: validation spread across a codebase makes it impossible to know whether all cases are truly handled. Parsing avoids this by stratifying programs into two phases — parsing (where failure from invalid input can happen) and execution (where the type system guarantees correctness).

Practical advice rounds out the essay: use data structures that make illegal states unrepresentable, push parsing to the boundary of the system, treat `m ()` functions with suspicion, avoid denormalized data, and don't be afraid to refactor toward better types. The ideas aren't new — they trace back to "write total functions" — but King's contribution is making the *process* of type-driven design communicable.
