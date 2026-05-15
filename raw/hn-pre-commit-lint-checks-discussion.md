---
title: "HN Discussion: Pre-commit lint checks: Vibe coding's kryptonite"
url: https://news.ycombinator.com/item?id=46560343
date_fetched: 2026-05-15
date_published: 2026-01-14
author: akshay326
---

# HN Discussion: Pre-commit lint checks: Vibe coding's kryptonite

Original post by akshay326 linking to getseer.dev (later redirected to civerify.com). 16 points, 27 comments. The blog post argued that pre-commit lint checks are the most effective guardrail against AI-generated code quality erosion.

## Top-Level Comments

### atrooo
Expressed fatigue with AI-generated blog posts about AI-generated code, questioning what the author gets out of it — "Upvotes?"

### Rantenki
Questioned the logic: if an AI assistant produces lint-violating code and can't process lint feedback to fix it, the AI tool itself is likely the problem. Drew a comparison to a junior dev who keeps breaking CI despite explicit instructions, noting you'd let that person go before probation ends. Asked rhetorically whether "the probation period for AI already expired."

### vaishnavsm
Warned TS developers about implicit `any` errors. Noted that LLMs frequently fix lint errors by using "explicit `any`s or `as any` casts" — which make the lint error vanish while preserving the actual logic bug. Even when told not to use `any`, LLMs may cast to `unknown` and narrow it to "a type that doesn't exist." Called these "valid code patterns" that LLMs tend to abuse.

### throwawayffffas
Recommended linters with autofix (like Ruff) and automatic formatters to reduce cleanup workload. Advised against over-fixing typing, noting "Python's duck typing is a feature not a bug." On duplicate code, suggested seeing at least two examples of a pattern before abstracting, to avoid abstracting "incidental duplication." Called coding agents "technical debt printers," but added you can "still pay it off."

### andsmi2
Described their pattern of forcing lint before push, requiring code coverage thresholds, and mandating all tests pass. Noted the same problems occur with human dev teams. Observed that "LLM actually listens to my rules a bit better than human devs" — and pre-commit/pre-merge checks enforce it regardless.

### OutsmartDan
Posed a philosophical question: if AI is writing **and** fixing all code, "does linting even matter?"

### cheapsteak
Suggested using `PostToolUse` instead of pre-commit hooks for lint checks — triggering on edit/write tool uses. For autofixable issues, the tool can format immediately. For type issues, suggested returning structured output that lets the edit complete but signals errors for the next turn.

### rcarmo
Stated that "linting and proper tests" are what enables them to use even simple models productively, "preferably writing the tests with a second model."

### rurban
Advocated for `-Wall -Werror` plus clang-format commit hooks. Argued that "proper languages cannot afford this kind of python or TS slop."

### seroperson
TL;DR summary: "Enable strict linting on CI, don't allow AI to change linting configuration."

## Selected Replies by akshay326 (OP)

- To Rantenki: agreed the tool is "broken" — "simultaneously stupid and smart in different ways" — but sees value in continuing to evaluate it.
- To vaishnavsm: agreed that detecting `any` usage as a guardrail is "simple yet effective."
- To throwawayffffas: called the "debt printer" metaphor apt, saying "I might steal it."
- To andsmi2: called it their "bitter lesson for the time being, unless claude gets eerily better."
- To OutsmartDan: observed that "LLMs try to cheat" and that left unchecked, "it tries to loosen the lint settings."
- To colechristensen (reply to OutsmartDan): noted that with linting, they "end up spending more tokens + time" than without.
- To cheapsteak: expressed curiosity about Rust's autofixable issues and their depth.
- To rcarmo: asked which simple models work well.
- To seroperson: admitted the TL;DR was accurate and "probably should've led with that instead of burying it 380 lines deep."
- To furyofantares (reply to seroperson): acknowledged the critique and linked a tweet version of the summary.
