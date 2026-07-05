---
title: "Ralph (Ghuntley's Technique)"
url: https://ghuntley.com/ralph/
author: Geoffrey Huntley
date_published: 2025-07-14
date_fetched: 2026-05-15
---

# Ralph - Geoffrey Huntley

Article introducing "Ralph," a technique for autonomous AI-driven software development using large language models in looped bash commands. The method emphasizes monolithic, single-task-per-iteration development guided by careful prompting and specification management.

## Core Concept

Ralph is fundamentally "a Bash loop" executing `while :; do cat PROMPT.md | claude-code ; done`. The technique leverages Claude Code (or similar tools without usage caps) to iteratively build software autonomously.

Key claim: "Ralph can replace the majority of outsourcing at most companies for greenfield projects."

The name references The Simpsons character Ralph Wiggum.

## Fundamental Principles

### Monolithic Architecture Over Microservices

Ralph operates as a single process in one repository per loop, avoiding the complexity of distributed agent communication. This contrasts with multi-agent approaches that introduce non-deterministic failures at scale.

### Single Task Per Loop

Each iteration must address exactly one item:
- "Only one thing per loop"
- LLMs prove surprisingly effective at reasoning about implementation priority
- Preserves limited context windows (~170k tokens for Claude)

### Deterministic Stack Allocation

Every loop should consistently allocate:
- A plan file (`@fix_plan.md`)
- Specification documents
- These function as persistent guides across context-limited iterations

## Prompt Architecture

### Example Production Prompt

The article provides the current prompt used for building CURSED (a compiler project):

"Your task is to implement missing stdlib and compiler functionality...Follow the fix_plan.md and choose the most important 10 things. Before making changes search codebase (don't assume not implemented) using subagents."

Key constraints within prompts:
- Use up to 500 parallel subagents for general work
- Restrict to 1 subagent for build/test validation (avoiding backpressure failure)
- Mandate test runs after each implementation
- Prohibit placeholder implementations
- Require documentation of test importance and reasoning

## Development Phases

### Phase 1: Generation

Code generation is now "cheap" and controllable through:
- Technical standard library specifications
- Detailed specification documents
- Guidance against non-standard patterns

If Ralph generates incorrect patterns, update the stdlib to steer future generations.

### Phase 2: Backpressure (Validation)

The critical bottleneck. Multiple validation mechanisms:
- Language type systems (Rust suggested for correctness, despite slow compilation)
- Security scanners
- Static analyzers (Dialyzer for Erlang, PyreFlag for Python)
- Unit tests focused on recently changed code

"The speed of the wheel turning matters, balanced against correctness."

## Key Techniques and Patterns

### Subagent Delegation

Ralph can spawn parallel subagents for expensive operations:
- Code searching via ripgrep
- File system exploration
- Documentation generation
- Planning via tree searches

This preserves primary context window for orchestration.

### Avoiding Duplicate Implementation

Common failure: ripgrep returns false negatives, causing Ralph to re-implement existing code.

Solution prompt: "Before making changes search codebase (don't assume an item is not implemented) using parallel subagents. Think hard."

### Self-Improvement Through Logging

Ralph updates `@AGENT.md` with discovered best practices:
- Correct command sequences
- Build optimizations
- Testing workflows

This creates institutional memory across loops.

### TODO List Management

The fix plan should:
- Prioritize remaining work by importance
- Identify TODO comments and placeholder implementations
- Search for missing functionality
- Be regenerated and discarded periodically when gone astray

"Ralph has three states: Underbaked, baked, or baked with unspecified latent behaviors (which are sometimes quite nice!)."

## Specific Implementation Guidance

### Writing Tests with Context

Tests must capture why they matter, since future iterations lose context:

```
@moduledoc """
Tests for the database query optimizer.

These tests verify the functionality of the QueryOptimizer module, ensuring that
it correctly implements caching, batching, and analysis of database queries to
improve performance.
"""
```

This helps Ralph decide whether future test failures represent genuine bugs or obsolete tests.

### Combating Placeholder Code

Models default to minimal implementations. The prompt includes:

"DO NOT IMPLEMENT PLACEHOLDER OR SIMPLE IMPLEMENTATIONS. WE WANT FULL IMPLEMENTATIONS. DO IT OR I WILL YELL AT YOU"

### Handling Broken Codebases

When Ralph breaks compilation:
- Assess: Is `git reset --hard` faster than salvage prompts?
- Create repair prompts for complex issues
- Consider farming difficult problems to other models (author used Gemini for compilation error planning)

## Real-World Results

### Cost Displacement

A case study cited: "Cost of a $50k USD contract, delivered, MVP, tested + reviewed with [@ampcode]...[$297 USD]." This represents ~168:1 cost reduction for a greenfield project.

### CURSED Project

Ralph is building a new programming language and compiler, including:
- Lexer and parser via tree-sitter
- LLVM code generation
- Standard library in the language itself
- Self-hosting capabilities

The language doesn't exist in Claude's training data, yet Ralph both created and programs in it.

## Limitations and Caveats

### Not Suitable for Legacy Codebases

"There's no way in heck would I use Ralph in an existing code base"

Ralph works best bootstrapping greenfield projects, targeting ~90% completion.

### Requires Senior Guidance

"Engineers are still needed. There is no way this is possible without senior expertise guiding Ralph."

The technique displaces junior/mid-level SWE roles but not senior architects.

### Eventual Consistency Model

Ralph requires "faith and belief in eventual consistency" -- accepting that the system will be chaotic during construction, resolvable through additional iterations.

## Broader Claims

### Post-AGI Territory

The author argues: "If models and tools remain as they are now, we are in post-AGI territory. All you need are tokens."

### Skills as Operator-Dependent

"LLMs are mirrors of operator skill." Success depends on the human operator's ability to craft effective prompts, interpret failures, and guide the system -- not the model's raw capability.

## The Signposting Metaphor

Huntley uses a playground analogy to explain how Ralph is tuned:

"Ralph is very good at making playgrounds, but he comes home bruised because he fell off the slide, so one then tunes Ralph by adding a sign next to the slide."

Signs are prompt instructions that nudge behavior in the right direction. Over time, signs accumulate. Eventually "all Ralph thinks about is the signs" — at which point you reset and start fresh with a clean prompt.

## Embracing and Resolving Defects

"any problem created by AI can be resolved through a different series of prompts."

Huntley describes waking up to broken codebases, using `git reset --hard`, or feeding compilation errors into Gemini to generate rescue plans that Ralph then executes.

## Additional Notable Quotes

- "All you need are tokens; these models yearn for tokens, so throw them at them."
- "When I hear that argument, I question 'by whom'? By humans? Why are humans the frame for maintainability?"
- "The models know what a compiler is better than I do. I just ask it."
- "Ralph has three states. Under baked, baked, or baked with unspecified latent behaviours (which are sometimes quite nice!)."

## Y Combinator Hackathon Result

A YC hackathon team ran a coding agent in a while loop and "It Shipped 6 Repos Overnight" — this became the RepoMirror project.

## Associated Resources

- GitHub: repomirrorhq/repomirror
- Related articles by author: "Deliberate Intentional Practice," "LLMs are Mirrors of Operator Skill," "From Design Doc to Code," "Autoregressive Queens of Failure," "I dream about AI subagents," "from Luddites to AI: the Overton Window of disruption"
