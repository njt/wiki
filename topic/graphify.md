# graphify

Turn a codebase into a knowledge graph. Fully multimodal -- drop in code, PDFs, markdown, screenshots, diagrams, whiteboard photos, even images in other languages. Uses Claude vision to extract concepts and relationships from all of it and connects them into one graph. Three outputs: interactive HTML visualization, markdown report, and queryable JSON.

---

## Key Quotes

> "Code files are processed locally via tree-sitter. Nothing leaves your machine."

## Key Themes

#tool #knowledge-graph #codebase-understanding #multimodal #visualization

The multimodal angle is the differentiator. Most code analysis tools handle... code. graphify handles 29 programming languages plus documentation, PDFs, images, video, and audio. The architecture diagram someone drew on a whiteboard? The PDF spec from the vendor? The screenshot of the bug? All become nodes in the same graph.

Code files stay local (tree-sitter parsing), video/audio uses local faster-whisper, and only non-code content goes to an API (Gemini, Claude, OpenAI, Ollama, or AWS Bedrock). The privacy boundary is drawn sensibly.

Graph analysis features -- identifying "god nodes" (most-connected concepts), detecting surprising cross-module connections, tagging relationships with confidence levels (extracted, inferred, ambiguous) -- turn this from a visualization into a diagnostic tool. The "research questions" it generates are a clever way to surface what you don't know about your own codebase.

47.6k stars is massive adoption for a code analysis tool. The team workflow (one person builds the graph, commits to git, teammates' agents access it) makes this practical for real teams.

## Critical Analysis

The auto-generated knowledge graph trades accuracy for coverage. Where [[lat.md]] requires human-maintained documentation (high accuracy, high maintenance cost), graphify extracts relationships automatically (lower accuracy, zero maintenance). The confidence tagging (extracted vs inferred vs ambiguous) is an honest acknowledgment of this tradeoff.

Git hooks for automatic rebuilding and conflict-free merge strategies show thoughtfulness about the team workflow. The graph becomes another artifact in your repo, not a separate system to maintain.

The interesting evolution would be combining graphify's automatic extraction with lat.md's validated annotations -- auto-generate the graph, then let humans mark which relationships are confirmed vs speculative.

See also [[lat.md]] for the human-maintained alternative, [[docmason]] for document-focused knowledge bases, and [[markitdown]] for converting documents into LLM-consumable format.

---
*Sources: [[summary/graphify]]*
*Last updated: 2026-05-14*
