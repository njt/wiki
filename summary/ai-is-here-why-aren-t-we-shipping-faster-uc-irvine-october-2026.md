---
url: https://www.youtube.com/watch?v=0wumMeZsNsI
title: "AI is here. Why aren't we shipping faster? (UC Irvine, October 2026)"
author: Christian Bird
date_fetched: 2026-11-13
date_published: 2026-10
topics:
  - agent-coding-workflow
  - ai-code-review
---

Christian Bird (formerly 16 years at Microsoft, now Dryden Technology Group) gave this 20-minute UC Irvine talk drawing on Microsoft-internal studies of what actually happens to software delivery once AI code generation arrives. His answer to the title question: generation got cheap, but everything downstream of generation — review, integration, testing, deployment — did not scale, and the people qualified to do that shipping work "did not multiply."

Key findings he reports:

- **Code volume explodes, releases don't.** A 500,000-developer GitHub study found commits up ~240% after AI adoption but releases up only ~30%. The gap is the un-shipped work piling up in review backlogs.
- **Context is the difference between success and production outages.** An engineer who understood a decades-old system got an agent to fix a decade-old quadratic-algorithm bug in a day; a PM who just said "fix this bug" broke production. The difference was feeding the agent the system's baked-in assumptions.
- **Non-engineers now ship code** — designers, PMs — and it's less clean. One manager: "I see code that is under my management changed by people I've never heard of before. And I wasn't even included on the PR."
- **"Systemic conversion strain"** (Yunha Cha's term): the cost of generating code fell, the cost of converting it into shipped software barely moved, but management expectations treated them as the same thing.
- **Models are not saving review.** A summer 2026 study of frontier models reviewing real code changes found they still let ~30% of issues through, and they find different things than humans (severe/reliability bugs vs. maintainability). Human+model is better than either. Microsoft's "lights-off software factory" goal works only for narrowly scoped, low-consequence PRs.
- **The productivity pressure paradox** (Courtney Miller): in a 27-paired-developer study (median 5.5 vs 33 days of AI use over 56 days), some developers are so pressured to deliver they lack the time to learn the tools that would save them time.
- Developers withhold full AI autonomy for two reasons: accountability (the 3am pager) and identity ("this is what I went to school for") — yet workload forces them to hand things over anyway.

His prescriptions: measure outcomes, not lines of code or PRs; scale verification (review "hasn't changed much since the 70s"); protect time to learn tools; and — the paper quote he calls profound — "we have a choice to set the boundaries deliberately through how roles, workflows and jobs are designed, or we can let the tool defaults set them with each release."
