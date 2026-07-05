# Summarize Meetings Skill

Harper Reed's Claude Code skill for monthly batch processing of meeting transcripts. The interesting part isn't the summarization itself -- it's that he expresses the entire workflow as a DOT digraph, making the processing pipeline visually inspectable and debuggable.

---

## Key Quotes

(Source file could not be fetched -- URL returned 404, possibly moved within Harper's dotfiles repo.)

## Key Themes

#skills #workflow #dot-graphs #meetings #person

The DOT digraph approach to expressing workflows is the real contribution here. Most Claude Code skills are linear instruction sequences. By encoding the workflow as a graph, Harper makes dependencies explicit: which steps can run in parallel, which must be sequential, where the data flows. This is the same insight behind [[Awesome Vibez]] member Braydon McCormick's speedrift-ecosystem, where pipeline DOT files serve as reusable blueprints.

Monthly batch processing is an underappreciated pattern. Most agent interactions are synchronous and conversational. Batch processing -- accumulate inputs over time, process them all at once on a schedule -- is how a lot of real knowledge work actually happens. You don't summarize each meeting the moment it ends; you review the month's conversations for patterns.

## Critical Analysis

The skill file itself wasn't retrievable, so analysis is limited to the annotation and the architectural pattern. The DOT digraph approach deserves wider adoption -- it's a natural fit for any multi-step skill where the agent needs to understand task dependencies.

The missing piece in most meeting summarization is action item tracking across meetings. A single meeting's summary is easy. Tracking which action items from last month actually got done requires persistent state, which is where [[session-analysis]] and similar tools could complement this.

See also [[agent-pr-replay]] for another approach to batch analysis of accumulated work product.

---
*Sources: [[summary/summarize-meetings-skill]]*
*Last updated: 2026-05-14*
