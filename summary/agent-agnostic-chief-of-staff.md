---
url: https://bquenin.dev/posts/agent-agnostic-chief-of-staff/
title: "qitt — an agent-agnostic chief of staff"
author: Benoît Quenin (bquenin)
date_fetched: 2026-11-11
date_published: 2026-11 (early November 2026)
topics:
  - personal-agents
  - agent-orchestration
---

Benoît Quenin describes qitt, his open-source (MIT, bquenin/qitt) proactive AI chief of staff rebuilt on top of T3 Code. It watches everything arriving on Slack and JIRA, and before the author opens anything it triages, routes, researches and drafts replies.

The first version, built on OpenClaw and living in Discord with long-term memory, failed in a instructive way: two months and ~280 commits in, he barely used it, because it was reactive ("I had to remember to go and talk to it") and located away from where he works. The rebuild changed two things: it moved to where his agents already live (T3 Code), and it became proactive — on a busy Friday morning it handled 42 inbound items before noon (25 routed to owning threads, 9 spun up research workers, 8 closed as acks).

Two hard rules shape the design. First, qitt never sends anything: Slack send tools are blocked at the tool level, not just prompted, and every one of the 166 worker drafts in three weeks was sent by a human. Second, it is built entirely from T3 Code constructs — the chief of staff is one pinned thread with an AGENTS.md charter, workers are ordinary threads, triggers are scheduled tasks (Slack webhook, JIRA webhook, a weekday morning sweep); only a small Slack bridge lives outside.

Three weeks of production numbers: ~940 incoming items handled, 367 closed needing nothing, 93 researched, 166 drafts written, ~4,000 messages in the pinned thread of which only 64 are his. Failures are fixed conversationally — complain in the pinned thread and qitt proposes or builds the fix. The closing thesis travels: your chief of staff should sit where all your agents already are, act before you ask, and never speak for you.
