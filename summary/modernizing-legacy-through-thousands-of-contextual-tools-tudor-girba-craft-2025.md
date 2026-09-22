---
url: https://gist.github.com/9857dcc8613863cd1e59b573a022952f
title: "Modernizing Legacy Through Thousands of Contextual Tools — Tudor Girba, Craft 2025"
author: Tudor Girba (speaker); ytx gist — summary + full transcript
date_fetched: 2026-09-13
date_published: 2025 (Craft 2025 talk; exact date not given in the gist)
topics:
  - software-engineering-craft
  - developer-tools
---

A ytx gist holding both a structured summary and the full transcript of Tudor Girba's Craft 2025 talk. The thesis: developers read code for more than half their working time — he says he has asked thousands of people, and cites studies — yet *how* they read is never discussed, so it is never optimized. He calls reading "the single largest expense we have in our job" and locates it, on a Wardley map, exactly where manual testing sat 25 years ago. The transformation that made testing win — composable micro-units compressing arbitrarily large functionality into a legible signal — is the template: "change the word testing into the word tools."

The case study is LifeWare, a Swiss insurance company with ~35M lines of code (~70M with artifacts) that has treated software engineering as a competitive advantage since it appeared as a case study in Kent Beck's 2002 TDD book: 4,000 tests took 20 minutes then; today it runs 150,850 tests in 18 minutes on an AWS cluster, and maintains ~2,000 contextual tools. Its modernization method is pixel-identical replay — rebuild a replacement system from the insurer's data alone and demand that replayed customer documents come out pixel-identical, which Girba calls "literally test redesigning the whole business." A live demo walks two problem paths from one test run: a business-side comparison failure (the CEO's signature changed; a domain-aware view highlights the pixel diff, travels into a business conversation, and supports one-click batch refactoring of the 36 same-cause failures) and an ops-side cluster investigation (worker spikes, task timelines, a staged five-minute wait he admits he cheated with).

The method has a name — moldable development: manufacture thousands of small, contextual tools, each answering one specific question, assembled at development time and allowed to be ephemeral ("I built enough of a tool to help me do the job"). Two roles structure the work: stakeholders (anyone with a stake) get the right to ask questions; facilitators manufacture answers as micro-tools. Guiding metrics are time-to-answer and, above it, time-to-the-interesting-question — and he closes on the admission that the real bottleneck is the latter. The environment behind the demo is Glamorous Toolkit (free, open source, 15 years of work), positioned as "the first large case study" rather than a product to adopt; the book *Rewilding Software Engineering*, written in the open with Simon Worley, lives at moldabledevelopment.com.

The gist's own fourth section is unusually good criticism: the cold-start problem (the case study is a company that never stopped modernizing — how do you bootstrap on *actual* legacy?), cost and ROI left as a black box, stack portability asserted but only shown on Glamorous Toolkit, no story for tool rot, staleness, or discovery at scale, no outcome data, a staged demo, and — conspicuous in 2025 — zero mention of AI or LLMs as a competitor or complement to manufactured views.
