---
url: https://arxiv.org/abs/2509.15834
title: "Automatic Layout of Railroad Diagrams"
author: Shardul Chiplunkar, Clément Pit-Claudel
date_fetched: 2026-07-25
date_published: 2025-09-19
topics:
  - software-engineering-craft
---

An EPFL paper presenting the first formal treatment of railroad (syntax)
diagram layout, with a working compiler. Railroad diagrams are widely used to
visualize grammars but have mostly remained hand-drawn; this work makes
automated layout principled rather than ad-hoc.

The authors define two languages: a **diagram language** with four
constructors (terminals, nonterminals, sequences, binary stacks) that captures
the conceptual structure, and a **layout language** with six constructors
(rail, space, station, hconcat, vconcat-inline, vconcat-block) that specifies
concrete shapes, positions, and connection points. The compiler translates
from the former to the latter through a three-step pipeline: **alignment**
(vertical positioning, styled à la Flexbox), **wrapping** (breaking sequences
across rows to meet a target width), and **justification** (distributing
horizontal space).

Wrapping is the central algorithmic challenge. The authors frame it as a
parametric optimization problem — maximizing content width, then minimizing
wrapping depth, then minimizing height — and avoid the exponential search
space with local greedy heuristics.

The implementation is ~1,065 lines of Scala (compiled to JavaScript via
Scala.js), produces SVG with CSS classes, and includes a web UI. An evaluation
against SQLite's 71 hand-drawn syntax diagrams shows that their layout
language can express most real-world variation; only a handful of cases
(ill-nested structures, stylistic inconsistencies) fall outside the
formalism.

The paper characterizes railroad layout as a "1.5-dimensional" problem —
between 1D text wrapping and 2D graph layout — because rows can split and run
independently before merging, or even turn backwards.

*Repository: github.com/epfl-systemf/librrd*
*Live tool: systemf.epfl.ch/etc/librrd/*
