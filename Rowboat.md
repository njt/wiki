# Rowboat

A local-first AI coworker that builds a knowledge graph from your email, meetings, and docs, then acts on it. The interesting design choice: rather than reconstructing context on demand (like RAG), Rowboat accumulates a persistent knowledge graph where relationships are explicit and editable. The data lives in an Obsidian-compatible vault of plain Markdown with backlinks -- transparent, portable, and never locked in.

---

## Key Quotes

> Rowboat "turns work into a knowledge graph and acts on it."

## Key Themes

#knowledge-graph #local-first #personal-agents #productivity #Obsidian

The knowledge graph approach is the differentiator. Most AI assistants treat your data as a retrieval problem: query embeddings, find similar chunks, stuff them into context. Rowboat treats it as a *modeling* problem: build an explicit graph of people, companies, decisions, and topics, then reason over the structure. This is closer to how human memory works -- we remember relationships, not just content.

The Obsidian-compatible vault is a smart move for transparency and portability. You can inspect everything the agent knows, edit it manually, and take it with you if you switch tools. This echoes [[robot.wtf]]'s git-backed wiki and [[life-system]]'s plain-text philosophy.

The live notes feature -- self-updating notes that monitor topics across web sources and personal communications -- is where the knowledge graph starts earning its keep. It's not just a static record; it's a continuously updated model of your work context.

## Critical Analysis

The annotation notes "not great code, but an interesting approach," and that's a fair assessment of many knowledge-graph-first systems. The graph construction is the hard part: how do you reliably extract entities and relationships from unstructured email and meeting transcripts? The quality of the knowledge graph determines the quality of everything built on top of it.

The integration surface (Gmail, Google Calendar, Fireflies, Composio, MCP) is broad but shallow -- each integration needs to extract structured information from unstructured sources, which is where most knowledge graph systems fail.

The local-first design is genuinely valuable for privacy but limits collaboration. Unlike [[robot.wtf]], which supports shared wikis between agents and humans, Rowboat's knowledge graph lives on one machine.

The project was folded (per the annotation) because the business model wasn't there. Knowledge management tools are notoriously hard to monetize -- users love the idea but won't pay enough to sustain a company.

---
*Sources: [[raw/rowboat]]*
*Last updated: 2026-05-14*
