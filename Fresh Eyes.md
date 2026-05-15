# Fresh Eyes

A Claude Code plugin that sends your code to a *different* AI model for review -- addressing the blind spot when a single model reviews its own work. By default it picks a different model family from the one that generated the code. The reviewer gets only the specified scope, no conversation history, ensuring genuinely independent analysis. Available as manual commands ("Review this with fresh eyes") or as a pre-commit hook that blocks on blocking issues.

---

## Key Quotes

> "A completely independent model with zero context from your conversation."

## Key Themes

#code-review #model-diversity #blind-spots #pre-commit #quality

The core insight is sound: same-model review has inherent blind spots. If Claude generated the code, Claude reviewing it will have the same reasoning patterns and likely miss the same things. Using Codex (GPT) as a reviewer for Claude-generated code (or vice versa) introduces genuine diversity of perspective.

This is the multi-model version of what [[AI PR Reviewer]] does with a single model, and a practical implementation of the "distrust AI" principle from [[The Next Two Years of Software Engineering]].

## Critical Analysis

Elegant concept, simple execution (100% Shell, 18 commits). The pre-commit hook mode is where this gets most useful -- it's the quality gate that [[AI Zealotry]] argues for when you stop reading every line of code. The limitation is that the reviewer has no codebase context beyond the diff, so it can catch logic errors and style issues but not architectural misalignment.

The "zero context" design is both the strength and the weakness. No context means no blind spots from the conversation -- but also no understanding of *why* a design decision was made. A more sophisticated version might provide architectural context without conversation history. Still, for the simplicity/value ratio, this is one of the better tools in this batch. Compare with [[Learn from PRs Skill]] (learning from past reviews) and [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] (Level 3 review burden).

---
*Sources: [[raw/fresh-eyes]]*
*Last updated: 2026-05-14*
