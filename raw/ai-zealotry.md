---
title: "AI Zealotry"
url: https://matthewrocklin.com/ai-zealotry
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# AI Zealotry by Matthew Rocklin

## Core Thesis
Experienced software engineers should embrace AI development tools like Claude Code because they amplify the high-level thinking that distinguishes senior developers, while automating the routine implementation work.

## Key Arguments

**Why AI Development Matters:**
The author contends that AI makes development more enjoyable by reducing tedious computer wrestling and enabling work in previously inaccessible domains like frontend development. Senior engineers are best positioned to use AI effectively because they can distinguish quality code from "AI slop."

**Why Not AI (Legitimate Concerns):**
- LLMs generate substantial amounts of poor code
- Reading generated code takes longer than writing it
- Self-written code builds deeper understanding
- Reviewing work can feel dehumanizing

The author dismisses these concerns by comparing AI adoption to the compiler transition—society gained substantially despite losing low-level understanding.

## Notable Quotes

"No, you're not too good to vibe code. In fact, you're the only person who should be vibe coding."

"Stop doing simple shit"

"Our ability to zoom in and implement code is now obsolete. Our ability to zoom out and think well is not."

## Major Practical Recommendations

1. **Minimize Interruptions Through Hooks** — Rather than relying on CLAUDE.md files (frequently ignored), implement hooks in Claude Code settings to enforce standards.
2. **Build Confidence Without Reading Code** — Heavy investment in tests and benchmarks. "Grilling" the AI about edge cases. Regular simplification reviews. Fresh-agent technical debt audits.
3. **Language Choice Shift** — Python's usability advantages diminish with AI assistance. Rust for computational work (with PyO3 bridges), TypeScript for frontend.
4. **Documentation Structure** — `plans/` for ephemeral development documents, `docs/` for durable references.

## Critical Insight on Cognitive Labor
Implementation is now "mostly free." Therefore, the thinking work—architecture decisions, edge case consideration, design clarity—becomes disproportionately valuable. Quality thinking deserves protected time, including "long walks."

## Conclusion
AI represents another rung on the abstraction ladder, similar to compiler adoption. The craft involves orchestrating feedback systems (tests, benchmarks, agent self-critique) that ensure generated code quality without requiring human code review of every line.
