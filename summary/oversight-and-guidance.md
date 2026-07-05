---
title: "Scaling LLMs to larger codebases"
author: "Kieran Gill"
source: https://blog.kierangill.xyz/oversight-and-guidance
date_published: 2025-07
date_fetched: 2026-05-14
type: blog
tags: [llm, coding-agents, guidance, oversight, prompt-library, one-shotting, codebase-health, verification]
---

# Scaling LLMs to larger codebases

**Author:** Kieran Gill (Blueberry Pediatrics)
**Date:** July 2025
**Series:** Part 3 of 3 on LLMs in software engineering

Part 1 covered "what LLMs and genetics have in common," Part 2 addressed "which areas LLMs do improve." This final part focuses on where to invest engineering resources to best leverage AI tooling.

## Summary

The central thesis: investments should flow into **guidance** (context and environment for LLMs) and **oversight** (skills to guide, validate, and verify LLM choices).

### Guidance → One-Shotting

**One-shotting** is when "an LLM can generate a working high-quality implementation in a single try." This is the most efficient mode of LLM programming. The opposite is **rework**, which "often takes longer than just doing the work yourself." Better guidance increases one-shotting opportunities.

### Building a Prompt Library

A prompt library collects documentation, best practices, codebase maps, and context engineers need to be productive. The iterative process: after each near-miss LLM output, ask "What could've been clarified?" and add that answer back.

Example: referencing multiple `@prompts/` files covering view conventions, testing practices, and API discovery — preloaded via `CLAUDE.md`.

Critical caveat: "**Read every line of generated code**. Just because you told an LLM to sanitize inputs, doesn't mean it actually did."

### Codebase Health as Context

The author recounts a peer at Meta saying Zuckerberg's automation claims weren't realistic because "their codebase is riddled with technical debt." This contrasts with the Cursor team: clean code principles apply "when you want it to be read by people and by models."

**Garbage in, garbage out:** "The utility of a model is bottlenecked by its inputs."

Two dipsticks for LLM literacy:
- Ask a peer to read unfamiliar code. If they struggle, the LLM will too.
- Ask an LLM agent to explain functionality you already know. Follow its search trail and document snags.

General principles: modularity, simplicity, good naming, encapsulated logic, consistency encoded in prompt libraries.

**Django-specific example:** The author's team uses `<app_name>_api.py` files as action-oriented entry points (e.g., `visit_api.handoff_to_doctor(user)`), so readers don't need full app context.

### Oversight Investment

"A 3-ton truck with a middle-schooler behind the wheel puts people in the hospital." Oversight targets **team**, **alignment**, and **workflows**.

Design/architectural skill growth: reading books/blogs/code, watching talks, **replicating masterworks** (citing Thorsten Ball's interpreter book), **reading code from leaders** (TLDraw, SerenityOS/Jakt).

Oversight also requires **temperament**, **alignment to values**, and **workflows**. Engineers need deep product understanding.

### Automating Oversight

Some design feedback can shift from human to computer, functioning as "bumper rails" that make it "impossible to land in the gutter." Safety checks protect abstractions.

Concrete example: enforcing the `_api` convention via AST-walking scripts that flag imports reaching into another app's internals.

### The Verification Bottleneck

As work volume increases, shipping becomes constrained by review capacity. Incomplete ideas:
- Lowering manual QA barriers
- Making test expression easy
- Encoding frequent PR feedback so LLMs can assist with review
- Baking security into framework defaults

## Key Quotes

1. "LLMs are choice generators."
2. "One-shotting" is when "an LLM can generate a working high-quality implementation in a single try."
3. "Read every line of generated code. Just because you told an LLM to sanitize inputs, doesn't mean it actually did."
4. "The utility of a model is bottlenecked by its inputs."
5. "Safety refers to the language's ability to guarantee the integrity of these abstractions"
6. "Design produces architecture. Architecture is a bet on the future."
7. "Modularity is an efficient way to build, since it relies on components that have already been tried and tested"

## References

- Mark Zuckerberg's automation claims (Forbes)
- Jack Danger's "Technical Debt Financing"
- Cursor team on clean code and LLMs (YouTube)
- Thorsten Ball's "Writing an Interpreter in Go"
- TLDraw V1 source (DrawSession)
- SerenityOS/Jakt parser
- Benjamin Pierce's "Types and Programming Languages"
- Alex Krupp's Django architecture post
- Will Larson's "Technical Strategy"
- Phillip Ball's *How Life Works* (modularity comparison)
