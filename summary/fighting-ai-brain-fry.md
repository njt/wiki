---
url: https://sergeykuhne.com/blog/fighting-ai-brain-fry
title: "How I'm fighting the AI Brain Fry"
author: Sergey Kuhne
date_fetched: 2026-11-09
date_published: 2026-11
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Sergey Kuhne diagnoses the cognitive failure mode of juggling many parallel coding-agent threads — "AI brain fry" — and proposes a fix: organise the work as a priority queue instead of a pile of open tabs.

The queue lives in git as markdown tickets next to the code they concern; the board's state is a committed snapshot, so nothing important lives in a chat window or in his head. An agent works the board in cycles: research new tickets, build ones tagged safe by a human, run tests and a browser check, verify deployments, and branch-and-ask for anything touching sensitive code. It never deploys to production.

Everything requiring human judgment funnels into a single "owner inbox" — a script reads the whole board and hands back a prioritised list of items waiting on a ruling. The reading surface is a Claude artifact, one page rebuilt from git every cycle, mirroring the board plus a review page per ticket; verdicts go back into git on the card. The page is only a viewer — git is the record.

The key trick is closing: pick one item, load its context, decide, and put it back in the queue — so the context closes with the ticket instead of hanging in working memory. He names the remaining challenges honestly: getting context served so loading a ticket takes a minute not twenty, and keeping tickets from looping while blocked on a ruling only he can give.
