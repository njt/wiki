---
url: https://www.parsonsmatt.org/2017/10/11/type_safety_back_and_forth.html
title: Type Safety Back and Forth
author: Matt Parsons
date_published: 2017-10-11
date_fetched: 2026-08-08
---

Matt Parsons distinguishes two directions for handling potential failure in a type system: pushing responsibility **forward** (using `Maybe`/`Either` to signal that a function might fail, leaving callers to handle it) and pushing responsibility **backward** (using restrictive types like `NonZero Int` or `NonEmpty a` so the function *cannot* fail in the first place).

The forward push is the familiar pattern: `safeDivide :: Int -> Int -> Maybe Int`, `headMay :: [a] -> Maybe a`. It's one-size-fits-all and easy to teach. The backward push is case-specific but more powerful: `safeDivide :: Int -> NonZero Int -> Int` requires callers to prove the divisor is non-zero before calling, and `safeHead :: NonEmpty a -> a` guarantees a result by construction. Haskell's `PatternSynonyms` extension lets library authors expose pattern-matching without exposing unsafe constructors.

Parsons draws the real payoff from a production refactoring: an `Order` type with `items :: [Item]` forced `Maybe` handling throughout the codebase because the list could be empty. Changing it to `items :: NonEmpty Item` — reflecting the business reality that every order has at least one item — purged all those `Maybe` values. The empty-list failure case moved to exactly two edge sites: JSON decoding (returning HTTP 400) and database reads (enforced via `INNER JOIN`).

The core insight: when you push type safety backward, the code encourages you to keep pushing it, until all the uncertainty lives at the system's edges — the API boundary and the database layer. Everything inside is pure, total, and simple. "By restricting our past, we gain freedom in the future."
