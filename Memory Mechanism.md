# Memory Mechanism

xAI's documentation on memory mechanisms in coding agents -- the best taxonomy I've seen of how agent memory should be structured. Five memory types (session, project, semantic, episodic, procedural), a critical distinction between instruction memory and learning memory, and a five-layer hierarchy from organization-wide rules down to role-specific scopes. This is the theoretical foundation for the practical memory systems in [[Context Rot]], [[mira-OSS]], and [[Claude's System Prompt]].

---

## Key Quotes

> "If these two types of memory are mixed together, system behavior often drifts over time."

## Key Themes

#memory #agents #architecture #context-engineering #best-practices

The five memory types map cleanly to what you see in practice:

- **Session memory** is the context window itself -- conversation history, recent tool outputs, execution plans
- **Project memory** is your CLAUDE.md and rules files -- architecture, conventions, build workflows
- **Semantic memory** is RAG over documentation and knowledge bases
- **Episodic memory** is past bug fixes and debugging strategies (the least implemented type in practice)
- **Procedural memory** is system prompts and workflow templates

The instruction vs. learning memory distinction is the single most important insight in the piece. Human-written rules (coding standards, security policies) should be stable; agent-accumulated learnings (user preferences, discovered patterns) should evolve. Mix them and the system drifts.

The five-layer hierarchy (organization > project > user > local > role-specific) mirrors how CLAUDE.md files actually work in practice: global instructions, project instructions, and local overrides.

The practical advice is sharp: keep memory files under 200 lines, split by topic, use concrete verifiable rules instead of abstract principles, define scope clearly.

## Critical Analysis

This is documentation, not research -- it describes best practices rather than evaluating them empirically. The five memory types are a useful taxonomy but don't address the harder questions: how should memories be promoted or deprecated? How do you handle contradictions between memory layers? What's the right ratio of instruction to learning memory?

The Roampal approach in [[Context Rot]] directly addresses the feedback loop problem (tracking whether retrieved memories actually helped), which this taxonomy doesn't. The five-bank architecture in Context Rot (working, history, patterns, memory_bank, books) maps roughly to the five types here but adds temporal dynamics (24-hour working memory, 30-day history decay).

The "memory files as contextual instructions, not enforced configuration" caveat is important and often misunderstood. Writing something in CLAUDE.md doesn't guarantee the agent will follow it -- it's a strong suggestion in context, not a hard constraint. Oversized memory files make this worse by diluting each instruction's influence.

For anyone building agent memory systems, this is required reading as a starting point. Then read [[Context Rot]] for the feedback loop, [[mira-OSS]] for first-person narrative memory, and [[robot.wtf]] for human-agent shared memory.

---
*Sources: [[raw/memory-mechanism]]*
*Last updated: 2026-05-14*
