---
title: "Systems Ideas That Sound Good"
url: https://hardcoresoftware.learningbyshipping.com/p/225-systems-ideas-that-sound-good
date_fetched: 2026-05-14
section: "Producing and Operating Software"
topics:
  - software-engineering-craft
---

Steven Sinofsky argues certain engineering patterns sound appealing but fail in practice approximately 9 out of 10 times.

1. Making Things Pluggable -- "the API is the behavior not the header file/documentation." True compatibility requires designing all implementations simultaneously.

2. Adding APIs to Become a Platform -- simply offering APIs doesn't guarantee adoption. Potential partners have their own businesses and customers.

3. Over-Abstracting -- premature abstractions often go unused (like Windows NT's unused layers), while post-hoc abstractions become maintenance nightmares.

4. Making Things Asynchronous -- modern frameworks handle this well, but "9 out of 10 times once you go outside a framework and think you can manage asynchrony yourself, you'll do great except for the bug that will show up a year from now that you will never be able to reproduce."

5. Deferring Security/Access Controls -- addressing access control after market launch guarantees failure.

6. Synchronizing Data -- Ray Ozzie's expertise: synchronization remains genuinely difficult, particularly with unstructured data.

7. Building Cross-Platform -- "fail as you diverge from the underlying platform or as you build capabilities that are expressed wildly differently on each target." Microsoft forked Office code in 1998 rather than maintain unified cross-platform builds.

8. Escape to Native -- frameworks maintain state that native calls bypass, creating unpredictable interactions.

Core lesson: solve problems from first principles rather than defaulting to familiar software patterns prone to failure.