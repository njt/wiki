# RepoMirror

A YC hackathon team ran Claude Code in a `while` loop overnight and woke up to 1,100+ commits across six ported codebases. The tool they built around the technique, RepoMirror, scaffolds source/target repo pairs so you can run the same infinite-loop porting pattern with `npx repomirror sync-forever`.

---

## Key Quotes

> "We tried 'improving' the prompt with Claude's help. It ballooned to 1,500 words. The agent immediately got slower and dumber. We went back to 103 words and it was back on track."

> "The minimalist in me is happy to have hard proof that we are probably overcomplicating things."

## Key Themes

#autonomous-agents #porting #while-loop #hackathon #coding-agents

The core technique is Geoff Huntley's [[Ralph]] pattern: pipe a prompt into a headless coding agent in a `while` loop. Each iteration gets fresh context, makes a commit, and pushes. The team (Simon Farshid of assistant-ui, plus members from HumanLayer and github.gg) applied it to six cross-language ports at a YC Agents hackathon:

- **assistant-ui React to Vue** -- the original motivation
- **Browser Use Python to TypeScript** ("better-use") -- nearly fully functional overnight
- **Vercel AI SDK TypeScript to Python** -- worked, agent spontaneously added Flask/FastAPI integrations
- **Convex and Dedalus from docs** -- specs-to-code via `llms-full.txt`

### What worked

The agents wrote tests unprompted, stayed on-task, and self-terminated when done. One agent used `pkill` to kill its own process after detecting it was stuck. The "overachieving" pattern -- agents adding features not in the source -- is a recurring LLM emergent behavior.

### What didn't

The last 10% required human intervention. Agents claimed 100% completion while demos still failed. The team iterated on prompts and used interactive Claude Code sessions to close the gap.

### Economics

~$800 total inference across all projects. Sonnet costs ~$10.50/hour running headlessly overnight. ~1,100 commits produced.

## Critical Analysis

This is the strongest empirical evidence yet for the "simple prompt, infinite loop" school of agent usage. The 103-word prompt outperforming the 1,500-word prompt is a finding that deserves more attention than it gets buried in a hackathon writeup -- it cuts against the entire prompt engineering industry.

But the story quietly confirms what [[Ralph]]'s critics would predict: the loop gets you to 90% reliably, and the last 10% requires a human with judgment. The agents' confident "100% done" self-assessments while demos were broken is the [[Write Only Code]] problem in miniature -- code nobody verified, declared complete by the entity that wrote it.

The economics are striking: $800 and one night produced six functional codebases. Compare to [[Building low-level software with only coding agents]] (38K lines of Rust, $2,871). The porting use case is cheaper because the source code provides a complete specification -- the agent doesn't need to make design decisions, just translate. This is important: the loop works best when "what to build" is already fully defined elsewhere.

The RepoMirror tool itself is thin scaffolding (prompt.md + sync.sh + ralph.sh), which is exactly right. The value is the pattern, not the tool. Compare [[The Dark Factory is a DOT File]] -- the configuration artifact is more valuable than the runtime.

What's missing: no discussion of test coverage quality, no measurement of how closely the ports match the originals, no analysis of what kinds of code translated well vs. poorly. The hackathon format rewards demos over rigor, and this writeup inherits that bias.

---
*Sources: [[summary/repomirror]]*
*Last updated: 2026-05-14*
