---
url: https://matklad.github.io/2022/07/04/unit-and-integration-tests.html
title: "Unit and Integration Tests"
author: matklad (Alex Kladov)
date_fetched: 2026-11-26
date_published: 2022-07-04
topics:
  - software-engineering-craft
---

matklad (rust-analyzer's lead) argues that the unit-vs-integration test distinction is confused and harmful — the terms mean different things in different communities, and most arguments about them are arguments about vocabulary. He proposes a two-dimensional replacement, adapted from the SWE book (Software Engineering at Google).

The first axis is **purity**: broadly, performance and stability. Test speed is categorical, not numerical — each step up a ladder of impurity (pure single-threaded computation → threads → time and disk → multi-process → distributed) costs about half an order of magnitude in runtime, and raises flakiness and fragility to unrelated changes. Borrowing functional programming's notion, pure tests do little-to-no IO and are independent of timings and environment; impurity has discrete notches (threads, time, filesystem, network, processes).

The second axis is **extent**: how much of the codebase the test exercises, potentially indirectly. Crucially, this is *distinct* from performance — running more code doesn't mean running slower; what matters for speed is whether persistence is backed by an in-process hash-map or an out-of-process database server. Extent is diagnostic (it reveals your architecture) but not worth optimizing for its own sake: mocking your *own* code to shrink extent reduces fidelity and makes refactors brittle.

The prescription: ruthlessly optimize purity; let extent be natural. A worked example from rust-analyzer — a trait-completion test — is trivially classifiable as 100% pure yet maximal extent, which is exactly the case the unit/integration vocabulary muddles. The SWE book's version differs mainly in calling the first axis "size" and replacing extent with "scope" (how large a bit you're actually testing), which matklad finds useful prescriptively but too fuzzy for classification.
