---
url: https://simonwillison.net/2026/Mar/6/ally-piechowski/
title: "Ally Piechowski: How to Audit a Rails Codebase"
author: Ally Piechowski (quoted by Simon Willison)
date_fetched: 2026-07-11
date_published: 2026-03-06
topics:
  - software-engineering-craft
---

Ally Piechowski proposes a set of diagnostic questions for auditing a Rails
codebase, organised by audience. Rather than a technical checklist, the
questions surface organisational and cultural signals: what people are afraid
of, what keeps slipping, and what promises have been quietly abandoned.

**For developers:** what area are they afraid to touch? When was the last Friday
deploy? What broke in production in the last 90 days that tests didn't catch?

**For CTOs and engineering managers:** what feature has been blocked for over a
year? Is there real-time error visibility right now? What was the last feature
that took significantly longer than estimated?

**For business stakeholders:** are there features that got quietly turned off and
never came back? Are there things you've stopped promising customers?

The questions function as a quick health scan — the answers reveal where fear,
brittleness, and technical debt have accumulated, regardless of what the commit
log or test suite says.
