---
url: https://ai.engineer/code/2026/talks/lauren-tan
title: "Slop into Gold — Lauren Tan (Potato) on shipping 3,000 PRs a month"
author: Lauren Tan ("Potato")
date_fetched: 2026-11-26
date_published: 2026
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Lauren Tan (aka "Potato"), working on the Grabbot client at SpaceX, gave an ai.engineer talk on turning "slop into gold": shipping 3,000 pull requests in a month with agents while code quality *improved*. Her thesis: the meta-job of a software engineer is maintaining or improving quality while massively increasing productivity — and it's all about trust, built not by prompting agents harder but by investing in their environment.

She lays out an inverse pyramid of investment: the bottom layers are deterministic (good API design, architecture, refactoring, lint rules, type systems), then verification (runtime and formal), and only at the thin top layer the judgment-requiring prose — rules, skills, agents.md. Her core rule: **every time you correct an agent, turn it into a deterministic check**, not another line of prompt. Your codebase is the agent's biggest input, so a pristine codebase means agents copy the good stuff — the "garden" analogy where constraints are the concrete that new agent code grows around.

Concrete examples from Dune, the Grabbot client framework ("Next.js for Electron apps"): banning `useEffect` in product code (nuance-requiring APIs generate slop), directory-structure context (`electron/renderer` vs `electron/main` imports forbidden by a dependency-graph analyzer), banning 99% of code comments (they go stale, are unenforceable, and agents use them to *justify* not doing the right thing), and lint rules with helpful error messages that tell the agent to fix the root cause.

Verification has replaced review as the trust mechanism: runtime verification skills with a "feature map" (a site map of the app from the user's POV), custom CLIs to cut token usage, TLA+ model checking of major user flows, Lean proofs for a custom virtualization library, and property-based UI testing. Most of her tokens go to verification — echoing Cindy's observation that 70% of daily tokens now do. The result: 11,000 PRs/month on the team, releases once a day, 227 contributors in October, revert rate steady at ~0.5%, and performance *improved*. The closing warning: if your inner loop produces slop, automating it just produces more slop — build the codebase and verification foundations before the factory.
