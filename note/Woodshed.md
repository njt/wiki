# Woodshed

Evals for Claude skills. An open-source framework for creating, testing, and iterating on skills through automated evaluation and refinement cycles. Create skill variants, run them against fixtures with custom evaluation criteria, and iterate based on results. "Create, run, rate, and iterate on your Claude Skills."

---

## Key Quotes

> "Create, run, rate, and iterate on your Claude Skills."

> "It runs Claude in yolo mode which can and will wipe your data."

## Key Themes

#evals #skills #claude-code #testing #quality

Woodshed fills a gap in the Claude Code ecosystem: there's been no systematic way to test whether skills actually work well. You write a skill, try it manually, and hope for the best. Woodshed makes this rigorous by defining evaluation criteria in `eval.md`, running skills against fixtures in batch, and having a dedicated evaluation agent score the outputs.

The idempotent runs design is practical -- re-running skips past results, so you can iterate on evaluation criteria without re-executing everything. Batch processing (up to 10 runs by default) accounts for the non-deterministic nature of LLM output.

Hosted on Tangled (a decentralized platform), which is philosophically interesting -- skill evaluation as a decentralized activity. Still alpha with 16 stars, so the community is small.

Connects to [[claude-code-config (Trail of Bits)]] (which defines skills but doesn't eval them), [[napkin]] (which improves skill execution through persistent learning rather than pre-deployment testing), and [[How to Write a Good Spec for Agents]] (where Osmani's "build self-checks" principle is what Woodshed implements for skills).

## Critical Analysis

The "yolo mode" warning is both honest and concerning. Running Claude with no permission checks to evaluate skills means the evaluation environment needs to be fully sandboxed, which adds setup friction. The tool is early-stage and the 16-star count suggests it hasn't found its audience yet. But the concept is right -- skills are code, and code should be tested. If the Claude Code skill ecosystem grows (it will), systematic evaluation becomes non-optional. Woodshed or something like it will be essential infrastructure.

---
*Sources: [[summary/woodshed]]*
*Last updated: 2026-05-14*
