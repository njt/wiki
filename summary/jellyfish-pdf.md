---
url: https://fion.ac/jellyfish.pdf
title: "Artificial Intelligence in the Firm: Bottlenecks in Software Production"
author: Fiona Chen & James Stratton (Harvard University)
date_fetched: 2026-10-08
date_published: 2026-08-04
topics:
  - ai-product-and-business
  - ai-code-review
---

A Harvard job market paper (Chen & Stratton, first version Jan 2026, current Aug 2026) using Jellyfish's engineering-analytics platform — ~300 million work events (GitHub, Jira, Google Calendar, HR) from 718 firms and ~726K workers, Jan 2021–Mar 2026 — to measure what happens when firms adopt AI coding assistants and agents. Identification is a staggered difference-in-differences on firm-level adoption timing, motivated by quasi-random variation in procurement process length (legal, security, piloting).

Both technologies raise coding output: assistants give small, mostly insignificant gains (12% lines of code, 9% commits, 5% PRs — only commits significant); agents give large, significant gains (30% LOC, 20% commits, 23% PRs). But the gains do not pass through: effects on Jira issue and epic resolution are small and insignificant, and agents' estimates rule out output gains above 12% — well under the 30% productivity gain. Employment effects are precise zeros (agents rule out a decline greater than 2.9% overall).

The mechanism is a code-review bottleneck. Review time per PR rises 49%, the share of PRs with changes requested nearly doubles, comments per PR rise 35%, and the share of workers doing review rises 14%. Engineers spend only 30–40% of their time writing code (validated against a 100-engineer Prolific survey), so the rest of the pipeline — review, testing, deployment — absorbs the extra code. The bottleneck persists even after firms adopt AI code review tools; human reviewers remain central.

They formalise this in a two-stage model (code writing + code review) where AI acts through two channels: a productivity channel (more code volume) and a bug-rate/quality channel (more verification effort per unit of code). The cleanest theoretical result: relative review-to-coding employment unambiguously rises — firms shift labour toward review and never toward coding as AI improves.
