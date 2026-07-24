---
url: https://arxiv.org/abs/2509.15834
title: Automatic Layout of Railroad Diagrams
author: Shardul Chiplunkar, Clément Pit-Claudel
date_fetched: 2026-07-25
date_published: 2025-09-19
---

# Automatic Layout of Railroad Diagrams

**Authors:** Shardul Chiplunkar and Clément Pit-Claudel, EPFL, School of Computer and Communication Sciences, Lausanne, Switzerland
**Submitted:** 19 September 2025
**Subject:** Computer Science > Programming Languages (cs.PL)
**License:** CC BY 4.0
**Pages:** 24 pages + 2 appendix + 3 references

## Abstract

Railroad diagrams (syntax diagrams) are a common visualization of grammars but mostly remain hand-drawn due to limited tooling. This paper presents the first formal treatment of railroad diagram layout with a principled, practical implementation. The authors define compiling from a *diagram language* (conceptual components and connections) to a *layout language* (shapes with sizes/positions), implementing line wrapping via optimization, vertical alignment, and horizontal justification per user policies.

## 1. Introduction

### 1.1 Contributions

1. A diagram language and layout language enabling precise specification of railroad layout as compilation.
2. Demonstration that the formalism captures most variation in hand-drawn diagrams.
3. A three-step compilation algorithm: alignment → wrapping → justification.
4. Wrapping framed as parametric optimization with practical heuristics.
5. An implemented compiler (Scala/Scala.js, 1065 SLOC + 260 SLOC rendering).

### 1.2 Background

The paper characterizes railroad layout as a "1.5-dimensional" problem — between 1D (text wrapping, code pretty-printing) and 2D (graph layout). Unlike text, "a row can split into two rows that independently follow the reading order until they merge, or turn backwards."

## 2. Formal Account

### 2.1 The Diagram Language

Four constructors:
- Terminal tokens: `"lbl"` (literal string)
- Nonterminal tokens: `[lbl]` (reference)
- Sequences: `(d ...)` — n-ary, zero or more subterms
- Stacks: `(pol d d)` — binary, with polarity `+` or `-`

Empty sequence `()` is denoted ε. Sequences are n-ary because association order doesn't affect layout. Stacks are binary because "having stacks be only binary is the simplest model that accounts exactly for all possible layouts."

### 2.2 The Layout Language

Six constructors: rail, space, station, hconcat, vconcat-inline, vconcat-block — with tip specifications (vertical, logical row, or physical proportion 0–1) and direction (ltr/rtl).

### 2.3 Layout Well-Formedness

Formal inference rules define width, logical rows, and connectability (up/down/both/neither) for each constructor.

### 2.4 Compilation Relation

The `diagram-of` function maps layouts back to diagrams. A layout ℓ of diagram d must satisfy equivalence and well-formedness.

## 3. Three-Step Layout Algorithm

1. **Alignment** — tip specifications and space placement for well-formedness. Cross-axis (vertical) positioning via align-items policies: top, center, bottom, baseline.
2. **Wrapping** — breaking sequences across visual rows to meet target width. Framed as optimization: greater content width > less/shallower wrapping > lower height (lexicographic). Local heuristics avoid 2^(k·(n−1)) exponential enumeration.
3. **Justification** — distributing available width among sublayouts. Three phases: min-content + gaps → proportional growth to max-content → flex-absorb + further growth.

Alignment and justification are styled as "à la Flexbox" — main-axis (horizontal) justification and cross-axis (vertical) alignment.

### 3.3 Implementation

- Language: Scala, compiled via Scala.js
- Repository: github.com/epfl-systemf/librrd
- Output: SVG with CSS classes and hierarchical structure
- Web UI: systemf.epfl.ch/etc/librrd/
- Default wrapping: minimizes max(0, w.max-content − target)² + 10·p_w(w)

## 4. Evaluation

### 4.1 Regular Expression and Backus-Naur Frontends

Mappings from regex/BNF to diagram language, including Kleene star as `(- () (rrd r))`. Notes tension between canonicity (equal representations for equal objects) and idiomaticity (convenient expression of common patterns).

### 4.2 Manual Layout in the Wild

Analysis of SQLite's 71 syntax diagrams: 45 perfectly expressible in layout language, 22 expressible in diagram language but must be laid out differently, 4 ill-nested requiring rewriting. Identifies stylistic inconsistencies (extra arrowheads, identical diagrams laid out differently "for no apparent reason").

Excluded cases from the formalism: Apple Pascal's ill-nesting, Crockford's visually ill-nested JSON layout, SQLite's vertical ε lines, and the hypothetical AlternatingSequence constructor.

## 5. Conclusions

Formal treatment of railroad layout is both possible and practical. The three-step approach produces layouts comparable to hand-drawn ones. Wrapping as optimization with local heuristics avoids combinatorial explosion. The diagram language captures most real-world railroad diagrams.

## Key Figures

- Figure 1: Four hand-drawn examples (Apple Pascal 1979, Wirth's Pascal 1970, SQLite 2024, IBM MQ 2025)
- Figure 2: JSON diagram in ECMA-404 layout + five alternatives from the tool
- Figure 3: Three-step pipeline visualization
- Figure 4: KAT theorem as railroad diagrams
- Figures 7-12: Formal constructor depictions, connectability, well-formedness rules
- Figures 14-17: Flexbox-style alignment/justification, dependency between them
- Figures 19-20: Wrapping type hierarchy, local optimality comparison

*Repository: https://github.com/epfl-systemf/librrd*
*Web UI: https://systemf.epfl.ch/etc/librrd/*
