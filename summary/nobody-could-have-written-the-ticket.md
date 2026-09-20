---
url: https://www.jamesdrandall.com/posts/nobody-could-have-written-the-ticket/
title: "Nobody Could Have Written the Ticket"
author: James Randall
date_fetched: 2026-09-20
date_published: 2026-08-01
topics:
  - agent-coding-workflow
  - specifications-as-the-product
---

James Randall threw away the entire AI opponent for his 4X strategy game Annhexation and rebuilt it from a new spec — and argues the version-two spec could only have been written *after* building and playing version one. The discarded implementation was the instrument that produced the better specification. Against the popular model of pointing agents at backlog tickets at volume, he argues the ticket is a hypothesis, not a contract: the expensive deciding already happened before it was written, and an agent that treats it as a contract will implement a bad hypothesis with perfect fidelity and never say so.

The dividing line he draws is not difficulty but whether the acceptance criterion exists outside a human head. Crash fixes and wrong calculations are fair targets for automation; design work is not, because nobody knows if a mechanic is interesting until it has been played. Ticket-to-merge is structurally waterfall — a single forward pass with no return path — and unlike human teams, agents don't cheat the process, so faithfulness becomes the flaw.

His positive claim is that agents make *iteration* cheap, not correctness: the gain is more hypotheses tested and killed earlier, and the hard ceiling is human judgement, which runs at human speed and doesn't compress. He closes with three confidence-tagged claims: the specifiable fraction of a backlog is a real ceiling (well under half for most product teams), value shows up as iteration count rather than throughput, and somebody has to keep playing the game.
