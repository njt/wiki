---
url: https://lethain.com/explorer-pattern/
title: "The Explorer Pattern for product invention and development"
author: Will Larson (lethain)
date_fetched: 2026-11-26
date_published: 2026-11-26
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

Will Larson names and codifies the "Explorer Pattern" — previously called the "Tech-led" or "Engineering-led" development model at Imprint: one individual or small team builds a functional implementation that minimizes cross-team impact, then reconciles the details with other teams afterwards, explicitly accepting substantial rework of the initial build.

The premise is that 2026 broke the old economics. Generating output — code, designs, analysis — is nearly free given appropriate constraints, and reworking software is extremely cheap, so process designed to avoid engineering rework (like protecting engineers' time by pulling them into problems late) optimizes the wrong constraint. His metaphor is database replication: the explorer is the primary, where writes are verified; other teams are secondary replicas who may be delayed but can't be wrong, so long as the working group keeps everything internally consistent. A concrete Imprint example: an engineering leader built Plaid financial-aggregator support as a low-burn side project, then cut "replica lag" to near-zero and shipped a much-wanted feature that was never on a roadmap.

The operational rules read like a discipline for exploring without getting lost: know your goal and sponsor (beyond ~3 weeks of effort, an unnamed enthusiastic stakeholder means you're "lost" rather than "exploring"); keep the working group at most five people who hold the whole project in their heads; publish progress in a public channel; make the narrowest viable changes to what exists; discard bad hypotheses immediately and implement the rest quickly; hold assumptions constant to avoid deadlock ("a lot of prework is just artificial precision"); hold space for being totally wrong; and ship all the way to production behind an internal feature flag from the start.
