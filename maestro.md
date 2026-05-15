# maestro

*Not to be confused with [[Maestro (UI Testing)]], the mobile/web end-to-end testing framework at maestro.dev.*

A multi-agent orchestration tool that organizes LLMs into roles mirroring a real development team: PM conducts requirements interviews, Architect breaks specs into stories and reviews code (but never writes it), and Coders pull from a queue, implement, test, and submit PRs. Coders terminate between stories; new ones spawn for new work. The goal is production-ready apps, not code snippets.

---

## Key Quotes

> "The goal is production-ready apps, not just code snippets."

## Key Themes

#multi-agent #orchestration #developer-tools #workflow

The role separation is the most interesting design decision. The Architect reviews and merges but doesn't write code, which prevents the common failure mode where the agent that designed the architecture also implements it and papers over its own design mistakes. Coders terminate between stories, which is a form of forced context freshness -- no stale assumptions carry over.

The operating modes reveal maturity: standard (GitHub), airplane (fully offline with Gitea + Ollama), Claude Code mode (uses CC as subprocess), hotfix (express path), and maintenance (automated tech debt). The knowledge graph in `.maestro/knowledge.dot` captures architectural patterns for consistency across stories -- a lightweight version of what [[Three Tier Memory]] does with its warm tier.

Connects to [[speedrift-ecosystem]] (which uses Workgraph for the task coordination that Maestro handles internally) and [[workgraph]] (which provides the task graph infrastructure that Maestro builds into its own orchestration). Also see [[ralph-ban]] and [[weft]] for simpler task-tracking approaches.

## Critical Analysis

The ambition is impressive -- this is trying to be a full software development team in a box. The risk is that the PM-Architect-Coder pipeline is too rigid for the way real development actually works, where roles blur and context sharing between roles is constant and informal. The "Coders terminate between stories" design prevents context accumulation but also prevents the kind of cross-story insight that makes a real developer effective. Worth watching, but I'd bet the sweet spot is simpler than this -- more like [[Compound Engineering]]'s "build systems that compound" than a full organizational simulation.

---
*Sources: [[raw/maestro]]*
*Last updated: 2026-05-14*
