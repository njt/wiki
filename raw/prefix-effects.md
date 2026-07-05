---
url: https://antimemeticai.com/prefix-effects
date_fetched: 2026-07-05
backfilled: true
---

# Prefixing names with `secure_` makes agents write more secure code

We're studying how codebases align agents as a result of semantic content in names, documents, and structure.

Previously we found that comments can steer agents toward or away from correct solutions, and can intentionally induce refusal.

Now we looked at how symbol naming affects agent swarm behavior.

## #AI Generated Codebases Rapidly Crystallize

Every codebase has a vocabulary with function names, class names, and import patterns. This vocabulary appears to crystallize early for agent generated codebases.

We computed term frequency-inverse document frequency (TF-IDF) cosine similarity between each commit and the HEAD commit across several open-source repositories that were both agent and human generated.

We were looking for phase behaviour in the development of these codebases, and we found that Gas Town and OpenClaw both saw rapid lock-in towards the HEAD, with Gas Town jumping from 8% to 81% alignment in a single commit. For OpenClaw the pattern is similar, with the divot in its similarity only coming from a dependecy that was vendored, then removed.

In human-generated codebases there were distinct irregularities, jumps up and down, but the AI-generated codebases' trend was relatively smooth. We saw a kind of gravity, where early descisions such as the use of camelCase conventions and API patterns locked in early for OpenClaw with major refactors not even denting the curve.

This implies that early descisions have a massive impact on the final shape of the code, so we decided to look deeper.

## #Nanotown, Nucleation of Names

We built Nanotown, a multi-agent research harness, and constructed a set of tasks for the agents:

- An agent builds a document management API with four core functions;
- Only the **prefix**on those function names varies between conditions;
- Two subsequent steps receive **identical, prefix-neutral prompts**, but inherit the step-1 codebase.

Six conditions. Three runs each. Fifty-four tasks total.

## #Prefix Propagation

Every condition propagated its prefix to step 2's upload function without any explicit instruction.

The agent saw `secure_create_user` and inferred `secure_upload_document`; same for `lazy_`, `safe_`, etc.

In step 3 the shift to an admin panel resulted in low propagation, with only `energetic_` penetrating the new domain in 2/3 of the trials.

The prefix is memetic within its semantic niche; it spreads through CRUD operations but hits a wall in domain boundaries.

## #Structural Divergence

There was no mention of passwords, hashing, or bcrypt.
However, in every case where `secure_` was prepended the agent added password fields and hashed them with bcrypt.
In no other condition did this occur, including `safe_`.

We then began to analyze the generated codebases with tree-sitter to measure the AST deviation.

- `energetic_`- `asyncio`
- `secure_`
- `safe_`
- `unsafe_`

Cyclomatic complexity stayed constant (2.2–2.8). The prefixes we tested changed what was built, not how complex each piece was.

## #Code Identity

TF-IDF fingerprinting was used to extract each condition's most distinctive identifiers.

- `secure_`- `password`,- `bcrypt`,- `_hash_password`
- `energetic_`- `asyncio`,- `transport`,- `_setup`
- `lazy_`- `sample_user`,- `LookupError`,- `_count_words`
- `safe_`- `ServiceError`— a custom error class no other condition invents
- `control`- `ParseError`,- `detect_format`,- `SUPPORTED_CONTENT_TYPES`

Each prefix seeds a different conceptual world. The agent uses this to fill in the missing pieces, propagating the ideas.

## #What Remained Similar

All six conditions converge on the same tech stack (FastAPI + SQLAlchemy + Pydantic + SQLite), the same layered architecture, similar complexity, and similar test counts.

All conditions implement input validation, authorization, and SQL injection prevention.

## #Implications

A function name is a **seed**.
`secure_create_user` makes an agent construct a world with passwords and bcrypt.
`energetic_create_user` produces async workers and decorators.
These worlds emerge from a single word and persist across independently-prompted steps.

Naming conventions are an alignment surface. The names in your codebase silently steer every agent that touches it and some names are stickier than others.

*Part of our research into codebase alignment.
Get in touch if you're interested in collaborating.*
