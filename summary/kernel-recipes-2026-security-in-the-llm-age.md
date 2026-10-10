---
url: https://www.youtube.com/watch?v=NnV_cWeoo5Q
title: "Kernel Recipes 2026 — Security in the LLM Age"
author: Greg Kroah-Hartman (Kernel Recipes)
date_fetched: 2026-10-10
date_published: 2026 (Kernel Recipes 2026 conference talk)
topics:
  - security-and-sandboxing
  - guardrails-and-feedback-loops
---

Greg Kroah-Hartman's Kernel Recipes 2026 talk is the Linux kernel security team's field report on the flood of LLM-generated vulnerability reports and patches arriving since Anthropic's Mythos announcement. He has the raw data behind the headline "79 kernel bugs": of those, 24 crashed with no usable report, 14 were not bugs at all, 3 were fabricated, 11 had already been publicly found and fixed by others, 4 were real-but-trivial bugs Anthropic itself fixed, and the final count of genuinely needed fixes was 20 — about one hour of kernel development at the current patch rate. His refrain throughout: do not panic; these are pattern matchers, not intelligent adversaries.

The real damage is not the bugs but the noise and the patch gap. Exploit lead time has inverted — from 63 days after disclosure to exploit to −7 days — because dumb-but-persistent bots chain minor issues that were never worth fixing individually. Half of LLM-generated patches that "look correct" are wrong (he tested this with six graduate students reviewing batches), the bots are sycophantic (scraping mailing lists to re-report already-fixed bugs to please the requester), and they produce signature patterns: mutex_unlock rewritten as mutex_destroy, bloated changelogs, gratuitous comments and flags, and decades-old kernel idioms from training data that Kokonel now rips to shreds.

The kernel's defenses are procedural: documented per-subsystem threat models (so bots can be pointed at "this is not a security issue"), requiring a patch as the first deliverable (which itself weeds out false positives), CCing maintainers, and the "assisted by" tag. His prescriptions for developers: push back, demand reproducers, ignore doom marketing, run local models and never upload non-public data (everything uploaded leaks — he cites Claude re-serving bugs found publicly by others and mathematicians' research resurfacing elsewhere). He closes with the fuzzer analogy: satan2/syzkaller felt apocalyptic too; we did the work, the flood ended, and rsync now scans clean. Verdict: a rough 12–18 months, then grind through it.
