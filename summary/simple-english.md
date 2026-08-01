---
url: https://github.com/AminBlg/SimpleEnglish
title: "SimpleEnglish — Write Like an Aerospace Manual"
author: AminBlg (Amin Baig)
date_fetched: 2026-08-01
date_published: 2026-07
---

A Claude Code skill that forces LLMs to write technical documentation in ASD-STE100 Simplified Technical English — the controlled language aerospace and defense manufacturers have used since 1983 for maintenance manuals. The core insight: STE's rules (max 20–25 word sentences, one word per meaning, simple tenses, active voice, no hedging, condition before command) are a near-perfect negative of every AI writing tell. Applying a 40-year-old aerospace standard produces text that is both technically rigorous and free of AI-generated clichés.

The project is a single 327-line SKILL.md file — pure prompt engineering, MIT licensed, zero dependencies — that works across Claude Code, Cursor, VS Code Copilot, and ~25 other harnesses that speak the Agent Skills open standard. It is not a library, tool, or API.

The skill operates in two modes (pragmatic and strict) and uses a classification-first approach: text is categorized as procedural or descriptive before any rules apply, since each type gets different sentence limits and verb forms. A pre-writing vocabulary constraint forces the model to pick one verb for a concept (e.g., check/verify/confirm) before drafting, preventing the synonym rotation LLMs default to. A non-optional four-part self-check runs on every output before delivery. "Untouchables" — code blocks, identifiers, CLI commands, quoted errors, product names — are explicitly exempt from STE rules, each counting as one word toward sentence limits, which is what makes the system practical for software documentation.

Supporting materials include before/after examples, use-case adaptations for eight document types (error messages, runbooks, incident reports, release notes, among others), and a deterministic regex-based linter (`ste_lint.py`) that catches ten categories of STE violations. A benchmark harness runs 96-generation matrices across models × scenarios and includes blind pairwise judging that controls for position bias. The project was developed using a TDD-style process: baseline model failures were recorded, rules were written to close each gap, and the skill was re-tested until agents passed.

The skill deliberately refuses to handle marketing copy, blog voice, or brand writing — it does one thing (make technical text unambiguous) and refuses everything else. It paraphrases STE rules rather than reproducing the copyrighted ASD dictionary, trading strict vocabulary enforcement for free MIT distribution.
