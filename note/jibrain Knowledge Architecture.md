# jibrain Knowledge Architecture

Joi's production knowledge architecture for a multi-agent system built on an Obsidian vault. A three-tier pipeline (intake → atlas → domains) with strict containment boundaries, frontmatter-as-contract, a formal routing decision tree, and seven-gate health audits. Describes a system in active daily use across multiple machines with Syncthing peer-to-peer sync and a team-facing read-only projection via Onyx.

---

## Key Quotes

> "Connecting 5 existing concepts is worth more than 5 new unconnected files."

Knowledge compounds through connection, not accumulation. This is the single most quotable line in the document and the thesis behind the entire reweave pass. System health is measured by connection density, not file count.

> "Small boundary violations compound silently until the system is too degraded to trust."

The argument for treating containment boundaries as structural requirements, not suggestions. Meeting preps leaking into concept directories, triage reports mixed with canonical knowledge, private contacts in synced partitions — each violation is minor in isolation but together they erode trust in search results.

> "A vault of 2,400 files is too large to read exhaustively."

This is the justification for the description field being "the single most important field." It enables filter-before-read: agents scan descriptions to decide which files to open. Without it, every query becomes an exhaustive search problem.

> "No single agent can corrupt the entire system."

The defense-in-depth argument for workspace boundaries. The curator can flood intake/ but cannot touch atlas/. The meeting extractor writes to intake/ but never to domains/. Each agent is confined to its lane.

> "Most knowledge systems only capture what you deliberately put in."

The pitch for agent observations (`intake/.observations/`). Agents log gaps, contradictions, and friction they notice during normal work. The system itself becomes a sensor, not just a store.

---

## Key Themes

#concept #tool #pattern

**Knowledge tiers as containment.** The three-tier structure (temporal/disposable intake, durable/authoritative atlas, deep/structured domains) is the architecture's load-bearing wall. Everything else — routing tree, triage gates, reweave pass — hangs off this distinction. The tiers are not organizational preferences; they're a structural defense against search pollution, staleness, queue blindness, and depth collapse.

**Frontmatter as contract.** YAML frontmatter isn't metadata — it's the API between agents and the vault. The description field specifically enables filter-before-read, which is the only viable search strategy at scale. Schema drift is tracked as a pipeline gate failure.

**Reweave over create.** The backward enrichment pass (find recently promoted files, then find and link existing files that should connect to them) is more valuable than creating new content. This inverts the typical ratio: more time on connections, less on creation.

**Multi-agent defense in depth.** Agents have explicit read/write boundaries. One agent's failure can't cascade. This is the same pattern as [[Elysia]]'s decision-tree tool constraints and [[klaw.sh]]'s namespace isolation, but applied at the filesystem level.

**Heartbeat as automation backbone.** The 15-minute collection and triage cycle with conservative auto-promote rules is the closest thing to a "set and forget" knowledge pipeline I've seen. It respects that intake/ is a queue that must be processed, not a dump that can be ignored.

**The router is a one-afternoon project.** The iblai-router (14-dimension weighted scorer, <1ms, 80% cost savings) was built and deployed within 24 hours of seeing Fred's architecture. This is the right level of ambition for routing: simple, auditable, no ML. The override mechanism (`[opus]` or `[haiku]` in message) is the pragmatic touch that makes it usable in practice.

---

## Critical Analysis

This is the most complete personal knowledge architecture for agents I've read. It's not a paper — it's field notes from someone running this daily. That's both its strength and its limitation.

**What makes it notable:** The document bridges two conversations that rarely meet. On one side, the [[Agent Memory and Context]] discussion about context windows, RAG, and retrieval. On the other, the [[LLM Wiki]] / personal knowledge management discussion about vaults, wikilinks, and synthesis. Joi's architecture shows how they're the same problem: an agent's memory is a knowledge base, and a knowledge base maintained by agents needs agent-native structure.

The three-tier pipeline is the right abstraction. Intake/ as a disposable staging area solves the problem every PKM system hits: the tension between "capture everything" and "find anything." The reweave pass is the most underrated idea in the document — most people optimize for creation throughput when connection throughput is the actual bottleneck.

The heartbeat system is pragmatic and boring in exactly the right way. No AI until 5+ items pile up or scheduled hours. Conservative auto-promote needs 6/6 gates to pass. Shell scripts and LaunchAgents, not Kubernetes. This is what production agent infrastructure should look like.

**Where it's incomplete:** The document is honest about what's missing, but two gaps deserve more attention. First, there's no adversarial verification — nobody is trying to break the system deliberately. Given that agent prompt injection is a real attack vector (see [[HackAPrompt Dataset]]), a system that trusts agent-written frontmatter is trusting an untrusted writer. Second, agent identity persistence is designed but not built, which means there's no agent accountability over time. If the meeting extractor corrupts an intake file, there's no trail.

The Onyx projection pipeline is clever but the manual corpus refresh is a real operational gap. "Automate from day one" is listed as a lesson learned, which means it wasn't done.

**What's missing from the document:** There's almost nothing about failure recovery. If Syncthing propagates a bad write from the sprite to the primary vault, how do you roll back? If the heartbeat promotor promotes a corrupted file, what's the undo path? The architecture is designed for correctness but not for recovery.

**Bottom line:** If you're building a personal agent system, this is required reading. If you're building a team knowledge base with agent contributors, it's a reference architecture. The routing decision tree and description field are the two highest-ROI changes you can steal immediately. The full Seven Gates audit is aspirational for most — start with gates 1, 2, and 4 as the document itself recommends.

---

*Sources: [[summary/jibrain-knowledge-architecture]]*
*Last updated: 2026-05-14*
