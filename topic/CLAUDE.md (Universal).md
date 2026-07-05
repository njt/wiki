# CLAUDE.md (Universal)

A minimal, token-efficient CLAUDE.md -- six rules for making Claude behave sensibly. Read files before writing. Be concise. Skip huge files. No pleasantries, no emojis, no em-dashes. And above all: "Do not guess APIs, versions, flags, commit SHAs, or package names." The annotation captures the essential truth: Claude will undoubtedly ignore these as context fills, because that's what happens in April 2026.

---

## Key Quotes

> "Do not guess APIs, versions, flags, commit SHAs, or package names."

## Key Themes

#claude-code #configuration #context-management #guardrails

This is a document about aspiration meeting reality. The six rules are all correct -- verification over assumption, directness over elaboration -- and they're all things Claude struggles with under cognitive load. The interesting question isn't whether these rules are good (they are) but whether any set of in-context instructions can survive the attention decay that happens as conversations grow long.

The answer, as the annotation notes, is basically no. Which is why [[Pre-Commit Lint Checks]] and [[claude-code-config (Trail of Bits)]] exist -- they move enforcement from "please remember this" to "you literally cannot proceed without satisfying this." See also [[Compound Engineering]] on building systems rather than relying on willpower.

## Critical Analysis

This is useful as a starting template but insufficient as a strategy. The CLAUDE.md is a suggestion; your linter is not. The real value here is as a reference for what to put in a CLAUDE.md when you're starting a project -- but anyone relying solely on this for quality is going to have a bad time. The fundamental lesson: prompt-level constraints degrade under load. Mechanical constraints don't.

---
*Sources: [[summary/claudemd]]*
*Last updated: 2026-05-14*
