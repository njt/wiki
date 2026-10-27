---
url: https://danluu.com/ambitious/
title: "We should be more ambitious with the software we build"
author: Dan Luu
date_fetched: 2026-10-28
date_published: 2026-10-28
topics:
  - agent-coding-workflow
  - ideas-and-culture
---

Dan Luu, prompted by Michael Malis's refrain "we should be more ambitious with the software we build," argues that coding agents have collapsed the cost of building things that were previously infeasible — and that most programmers are still budgeting with a pre-agentic cost model. Malis's pgrust (a Postgres rewrite) is the example: easily dismissed as vibeslop, but its testing is thorough enough to have uncovered real Postgres bugs, so even a failed attempt durably improves the ecosystem.

Luu demonstrates the claim with his own projects: a regex engine, a ripgrep fork, and a Rust interpreter that speeds cold builds 50x — things that would have taken him a year or more, done in low single-digit hours with no domain expertise. His point is not that "everything just works"; rather, each more capable model lets him attempt more ambitious work where models still struggle, and the gap between what a model can implement and what it can verify unaided is widening.

He is careful about the limits. Yossi Kreinin's counterpoint (appended) argues most people should be *less* ambitious, because they lack pass/fail criteria and can't reason about giant black boxes. Luu's rejoinder: slop elsewhere is no reason not to build well yourself, and many problems admit partially self-checking setups — fuzzing, property-based testing, consistency checks across contradictory specs — that let a person with little domain knowledge verify agent output cheaply. A long appendix of examples (board-game AIs, throwaway games, Geoghegan's Neovim formatter, spare-time tools, Anthropic's internal GitHub replacement) grounds the thesis in "what you can do today, without unlimited tokens."
