---
url: https://lalitm.com/post/find-problems-staff-engineer/
title: "Find Problems Worth Working On"
author: Lalit Maganti
date_fetched: 2026-09-13
topics:
  - software-engineering-craft
  - ideas-and-culture
---

Lalit Maganti (Perfetto engineer at Google) answers a mentee's question about making the jump to staff engineer: how do you find problems worth working on, rather than just solving the ones you're assigned? His answer is not blocking out calendar time to "think strategically" — that produced nothing for his mentee either. Instead he acts "like a sponge": absorb the stream of day-to-day problems people mention, ask what they're actually trying to accomplish rather than taking solution requests at face value, sit with teams through their workflows and bugs, and seek out people who see more of the organization than you do.

The method has four movements. **Absorb problems, not requests** — users describe wishes, not root issues; dig until you understand the underlying need. **Let problems accumulate** — eager requests are not evidence of importance; he has built eagerly-requested features that went unused, so now he lets candidate problems sit, and revisits them only when they recur independently or reveal a shared shape. **Find the common shape** — with Perfetto, years of small UI requests (pinned tracks, default zoom, custom aggregations, bookmarklet workarounds) collapsed into one realization: teams wanted to personalize the UI without imposing their choices on others, so the real feature was UI extensibility. But a common shape "is only a hypothesis and elegance is not evidence" — a later attempt to unify trace-sharing and repeated-query problems behind one caching system had to be split in two, and both halves shipped. **Pressure-test before building** — escalate commitment with confidence: send small changes directly, build throwaway prototypes when unsure, and commit to the full RFC-and-socialization effort only for big ideas you're convinced by, while staying willing to stop or park the idea.

The payoff compounds: genuinely helping with people's problems makes them come to you earlier and bring you into more conversations, widening your view and making the next pattern easier to see. Trust from past calls means his judgment about what matters gets weight, so he can influence roadmaps without owning every implementation. He explicitly rejects the staff-engineer-as-meeting-coordinator stereotype: conversations are inputs to what he builds, not the end result. Caveat: this comes from infrastructure and developer-tools work at large companies with bottom-up autonomy; a top-down environment may leave little room for it.
