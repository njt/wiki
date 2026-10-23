---
url: https://danluu.com/sounds-easy/
title: "It Sounds Easy"
author: Dan Luu
date_fetched: 2026-10-23
topics:
  - software-engineering-craft
  - ideas-and-culture
---

Dan Luu takes on the perennial internet claim that big software companies are bloated — that some small team could rebuild Google, or that a product obviously requiring thousands of engineers can't really need them. Using search as his worked example, he argues that even the "easy" parts are hard: indexing trillions of items in real time under load is non-trivial with off-the-shelf tools like Lucene, and ranking — which he considers the harder problem — is only harder still.

From there he generalizes. Businesses that care about profit keep hiring engineers for optimization and features until the marginal engineer no longer pays for herself, and features are far more complex than outsiders see: internationalization, tokenizing Chinese, handling Arabic's mixed directions, security. Then comes the deeper point: organizational problems dwarf technical ones. Human communication loses over half its messages and has latency measured in months, so inefficiency is structural, not a bug to be removed.

The closing analogy is the essay's spine: error handling is more work than the happy path, and everything outsiders dismiss as not "real" work — billing, sales, research, i18n — is more work than the core service. A correction note at the end, where Luu retracts a back-of-envelope calculation after failing to reproduce the underlying benchmarks, doubles as a quiet demonstration of his own standards of evidence.
