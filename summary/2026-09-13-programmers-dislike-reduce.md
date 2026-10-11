---
url: https://web.archive.org/web/20261003224700/https://evanhahn.com/posts/2026-09-13-programmers-dislike-reduce/
title: "Anecdotally, programmers dislike 'reduce'"
author: Evan Hahn
date_fetched: 2026-10-11
date_published: 2026-09-13
topics:
  - software-engineering-craft
---

Evan Hahn documents a small social phenomenon from his own code review experience: `map` and `filter` sail through peer review without comment, but patches containing `reduce` reliably draw "this part is hard to read" feedback, even though Hahn personally prefers it.

He offers five candidate explanations: `reduce` is genuinely harder to read; it's less familiar; it can have worse performance than alternatives; it's less elegant in the languages he uses (JavaScript, Python, Swift) — notably he got no such pushback during a stint in Clojure, where fold-style code is idiomatic; or the trend isn't real and he's pattern-matching on noise.

His usual response is to just swap `reduce` out and move on — he doesn't care enough to fight. He also observes the pushback has lessened recently, which he attributes to code review becoming less thorough. The piece ends by soliciting whether others see the same.
