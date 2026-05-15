---
title: "Slowing the Fuck Down"
url: https://mariozechner.at/posts/2026-03-25-thoughts-on-slowing-the-fuck-down/
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Thoughts on Slowing the Fuck Down - Mario Zechner

## The Core Problem
Coding agents have enabled rapid code generation without corresponding quality controls, leading to brittle, unmaintainable systems. Removing humans from the development loop eliminates natural bottlenecks and learning mechanisms.

## Key Issues

1. **Compounding Errors Without Learning**: "An agent has no such learning ability. At least not out of the box. It will continue making the same errors over and over again." Unlike humans who learn from mistakes, agents generate code smell and errors at scale without self-correction.

2. **Elimination of Healthy Friction**: Removing humans from loops removes pain signals that normally trigger code quality improvements. Booboos accumulate unsustainably until the codebase becomes unmaintainable.

3. **Local Decision-Making Creates Complexity**: Agents lack comprehensive codebase visibility, leading to duplication, unnecessary abstractions, and cargo-cult architecture patterns that compound into "unrecoverable mess[es] of complexity."

4. **Search Limitations**: "The bigger the codebase, the lower the recall" of agentic search tools, preventing agents from finding relevant existing code and amplifying duplication problems.

## Evidence Cited
- AWS outage allegedly involving AI-assisted code changes
- Microsoft's 90-day code reset initiative following outages
- Widespread reports of products built entirely with agent code exhibiting severe quality issues
- Multiple anecdotal accounts of companies "agentically coded into a corner"

## Recommended Approach

**What Agents Should Handle:**
- Scoped, self-evaluating tasks
- Non-critical tooling and ad-hoc software
- Research and exploration activities with clear metrics

**Human Responsibilities:**
- Architecture, API design, and system definition decisions
- Final quality gating and code review
- Setting generation rate limits aligned with review capacity
- Maintaining understanding of system design and rationale

## Key Conclusion
"Slowing the fuck down" by introducing deliberate friction into development preserves human agency, enables learning, and ultimately produces superior systems. "This is where your experience and taste come in, something the current SOTA models simply cannot yet replace."
