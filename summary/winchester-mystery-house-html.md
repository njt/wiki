---
url: https://www.dbreunig.com/2026/03/26/winchester-mystery-house.html
title: "The Cathedral, the Bazaar, and the Winchester Mystery House"
author: Drew Breunig
date_fetched: 2026-09-13
date_published: 2026-03-26
topics:
  - ideas-and-culture
  - agent-coding-workflow
---

Drew Breunig extends Eric Raymond's 1998 cathedral/bazaar dichotomy with a third model for the agentic era: the *Winchester Mystery House*, named for Sarah Winchester's 160-room, 2,000-door San Jose mansion — built with unlimited funds, no training, and pure personal passion. AI has made code cheap the way the internet made communication cheap, and developers are responding like Sarah: building idiosyncratic, sprawling, fun tools for themselves.

The mechanism is economic. Claude commits now average ~1,000 net lines added per commit — roughly two orders of magnitude above human benchmarks (antirez calculated his own Redis output at ~29 LOC/day). But everything *else* — feedback, review, coordination — still costs the same. The only feedback that moves at the speed of AI-generated code is your own, so the feedback loop collapses into one person: near-zero latency, throughput of one. Steve Yegge's Gastown, Jeffrey Emanuel's Rust "FrankenSuite," and Gary Tan's gstack are all Winchester Mystery Houses.

The bazaar isn't abandoned, it's drowning: agent-written PRs are slamming maintainers (curl ended bug bounties; GitHub added a feature to disable PR contributions), machine-speed implementation hitting coordination infrastructure built for human speed. Breunig inverts Linus's Law: we need eyeballs to find bugs *before they reach* the software, not in it.

Three lessons: (1) the models coexist — OpenClaw is both a bazaar success and a foundation for thousands of personal Mystery Houses; (2) don't sell the fun stuff — the commons (or companies) should own the boring, critical, disastrous-failure-mode things (Sarah Winchester bought off-the-shelf plumbing but hired craftspeople for stained glass); (3) the new limit is attention — the internet made coordination cheap, agents made implementation cheap, and nothing yet makes attention cheap, so good ideas surface nowhere and maintainers drown.
