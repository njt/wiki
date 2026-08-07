---
url: https://stng.substack.com/p/rewrite-all-the-code-all-the-time
title: "Rewrite All the Code, All the Time"
author: Adam (stng.substack.com)
date_fetched: 2026-08-07
---

# Rewrite All the Code, All the Time — Précis

Adam argues that production-ready code should stop being treated as a scarce resource. Within a few years, the cost of regenerating significant codebases will drop to SaaS-subscription levels — and the durable artifact will be the specification, not the implementation. When new security requirements or algorithms emerge, we should be able to regenerate all code to comply at near-zero cost.

The mainstream path — AI code generation from natural-language requirements — won't get us there. Natural language is inherently ambiguous, and the EARS (Easy Approach to Requirements Syntax) format illustrates the problem: formal keywords wrapped around freeform phrases leave ambiguity exactly where behaviour is decided. The history of computing teaches that abstraction levels rise steadily (flowcharts → high-level languages → formal specs), and the next step is replacing natural-language spec formats with unambiguous formal languages.

The key enabler is **formal methods**. Only mathematically rigorous specifications can support *truly automatic* generation without human oversight — regeneration should be as push-button as recompilation. A thought experiment: if one contractor writes the spec and a succession of adversarial implementers each build to it, the spec becomes the moat, and conventional codebases lose their competitive value.

A reader response adds two extensions: (1) the missing third artifact — the record of what was checked, under which assumptions, with which exclusions — which may carry cost that migrates rather than disappears; and (2) the version-control question for specs: who contributed which constraint, and what is it worth?
