# napkin

A Claude Code skill that gives the agent persistent memory of its mistakes via a per-repo markdown scratchpad (`.claude/napkin.md`). "Baby continual learning in a markdown file." The agent logs its mistakes, user corrections, environment surprises, and successful approaches. Performance noticeably improves by sessions 3-5 as the agent starts anticipating issues before being corrected.

---

## Key Quotes

> "Baby continual learning in a markdown file."

## Key Themes

#memory #learning #claude-code #skills #mistakes

The genius of napkin is its simplicity. No database, no embeddings, no retrieval pipeline -- just a markdown file that accumulates knowledge over time. The agent reads it at the start of each session and writes to it whenever it learns something. This is the minimum viable memory system.

The per-repo design is correct: what you learn debugging a React app is different from what you learn on a Go CLI tool. The choice to make the napkin committable (shared with collaborators) or gitignored (personal) is thoughtful.

This is the simplest implementation of the problem that [[Three Tier Memory]] solves at the architectural level. napkin is the hot tier and nothing else -- 660 lines of always-loaded context containing your mistakes and preferences. For small projects, that's enough. For 108K-line C# systems, you need the full three-tier approach.

Also connects to [[engineering-notebook]] (both make agent sessions leave traces) and [[Slowing the Fuck Down]] (napkin is one answer to "agents don't learn from mistakes" -- force them to read their mistake log).

## Critical Analysis

The improvement by sessions 3-5 is the key claim, and it's plausible -- a markdown file of "don't do X, do Y instead" is exactly the kind of concrete instruction that LLMs follow well. The risk is that the napkin grows without curation until it's full of contradictory or outdated advice. There's no mechanism for pruning or prioritizing entries. A napkin that says "always use approach A" and later "never use approach A" will confuse the agent. Manual curation is needed but not enforced. Still, for the cost of zero infrastructure and one markdown file, the ROI is extraordinary.

---
*Sources: [[summary/napkin]]*
*Last updated: 2026-05-14*
