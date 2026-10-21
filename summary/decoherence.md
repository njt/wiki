---
url: https://maevyn.codes/decoherence
title: "The Decoherence Trap"
author: maevyn (maevyn.codes)
date_fetched: 2026-10-21
date_published: 2026-10
topics:
  - software-engineering-craft
  - ai-research-and-models
---

maevyn's essay names a third kind of tech debt, **decoherence**: the accumulation of locally valid but mutually incompatible models, paradigms and hand-crafted patches in a system, such that the cost of change is dominated by translating among them. It is the worst kind of debt because it grows with *change*, not time, compounds exponentially (more decoherence makes the next change sloppier and more complex), and is invisible to management in a way broken pipes in a wall are not. The Ship of Theseus and the plumber-replacing-pipes thought experiments carry the argument; a small interactive simulation shows the emergent "Sweet Spot" curve where actor coherence trades speed against system complexity.

The essay's central charge against the AI coding moment is that agents are currently a decoherence accelerant: the Harvard Bottlenecks study (more code, same finished work, 49% longer PR reviews), the CMU Cursor study (velocity spike that fades, complexity that stays), and a 300k-commit audit (~1 in 11 AI commits with a quality issue) all point the same way. maevyn then turns the concept against the Software Intelligence Explosion thesis: since good software is *coherent* software and coherence can only be validated against real-world feedback — new problems, evolving requirements, shifting ecosystems — an AI cannot measure whether it is actually improving coherence from inside the data-center bubble, which lands us back at the Goodhart Singularity rate limit. Mathematics runs into the same wall: producing true statements in a vacuum is not the same as knowing which ones matter, as the Fields-medalist declaration argues.

The closing warning is not superintelligence but **Decoherence Collapse**: a software ecosystem so fragmented and agent-dependent that we cannot maintain it ourselves and agents cannot back us out. The remedy is deliberately unfashionable — coherence work (unifying paradigms, finishing migrations, open-source style consolidation of defaults), which only ever happens when systems change, and so can always be restarted.
