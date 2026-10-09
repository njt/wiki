---
url: https://jessewaites.com/blog/post/i-pointed-ai-at-400-years-of-archives/
title: "I Pointed AI at 400 Years of Archives"
author: Jesse Waites
date_fetched: 2026-10-10
date_published: 2026-10-08
topics:
  - agent-orchestration
  - guardrails-and-feedback-loops
---

Inspired by Benjamin Breen's dodo discovery in the GLOBALISE transcriptions of the Dutch East India Company archive (4.35 million pages), software engineer Jesse Waites built an AI research pipeline to hunt for other overlooked historical records — and found a previously unrecorded 1812 meteorite fall in Maharashtra, three Javan rhinoceroses shipped (and died) en route to the King of Kandy in 1738–1740, and three volcanic eruptions (Gamkonora 1722, Ciremai 1712, Slamet 1780) missing from the Smithsonian's Global Volcanism Program.

The pipeline is a tiered delegation chain: embedding search over 5.7 million passages (semantic, not keyword, because 17th-century Dutch spelled "rhinoceros" fifteen ways), then a cheap "System One" decision model called Jev as the first-pass filter — reading 59,000 elephant mentions for about three dollars — then Claude Haiku for close reading and translation, then a Claude Code agent checking each transcription against the scan of the original handwritten page, with claims finally checked against the specialist catalogues.

Two methodological rules carry the piece. First, positive controls: before trusting a null result, the pipeline had to rediscover known events (Breen's dodo, Laki 1783, Tambora 1815) — "testing a metal detector by burying your own wristwatch." Second, honest scoping: Waites insists this was AI-assisted, not autonomous — the human steered the questions, picked the leads, and knew when to stop. Failures are reported too: the hunt for the "Unknown" eruption of 1808/09 came up empty, with the untranscribed logbook margins flagged as the next frontier. Waites is open-sourcing the workflow as a toolkit, Antiquity.
