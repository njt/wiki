# OzBrain

A commercial "shared brain" that every AI agent you use can read and write — one structured, cross-platform knowledge base behind Claude, ChatGPT, Cursor, Claude Code, and any client that speaks the connector protocol. The thesis: platform memory keeps scraps and thin summaries in per-product silos; OzBrain holds the work itself, and lets agents do the maintenance humans abandon.

---

## Key Quotes

> "One shared brain that Claude, ChatGPT, Cursor, and every AI can read and write. It structures what you know so agents read only what they need, and means you never explain yourself twice."

The product thesis in two sentences. Note the two halves that do different work: *cross-platform* (one brain, many agents) and *selective* (structured so agents read only the slice they need). Most memory tools in this wiki solve one or the other — [[Coding Agents Continuity Not Memory]] solves continuity within one tool, [[The Knowledge Chipper]] documents the cross-tool rebuild tax — but OzBrain's pitch is that the two problems are the same problem.

> "Copy a brief into Claude. Paste it into ChatGPT. Drop the same .md into Cursor. Update one copy, forget the others, and watch them drift. That is the job you are stuck doing: ferrying context between tools that do not share a source of truth."

The clearest statement of the pain being sold against. "The current version is wherever someone last saved it" is the same observation that drives [[The Knowledge Chipper]]'s multi-model tax and [[Context Graphs]]'s "systems of record with the reasoning stripped out" — but framed as a *product*, not a research problem.

> "Humans abandon wikis because the maintenance burden grows faster than the value."

Attributed on the page to an "OpenAI co-founder · Anthropic R&D" (unnamed), who "published the blueprint for a knowledge base your agents build and maintain." This is the load-bearing assumption of the whole product: the reason human wikis die is precisely why *agent*-maintained wikis might live. It echoes the [[LLM Wiki]] pattern — the LLM handles the bookkeeping while humans curate — and is the direct rebuttal to [[Memory Is a Mistake]]'s sixth failure mode, the maintenance burden.

> "Platform memory keeps scraps and summaries. OzBrain holds the work itself."

The positioning table (preferences/scraps/thin summaries vs. "what you know, what you are working on, and what is next") draws a bright line: a "small, capped profile" that "lives inside one chat product" versus "a linked library that grows with your work" and "routes to the article the task needs." The claim is that memory should hold *artifacts* (projects, decisions, research), not *preferences*.

---

## Key Themes

- **#tool** — A hosted, cross-platform shared brain: one MCP URL, connected from the Claude/ChatGPT connector menu, not "a service you code against" (the explicit Mem0/Supermemory contrast).
- **#concept** — Context ferrying as the job to eliminate: the same brief copied, pasted, and drifted across tools because none share a source of truth.
- **#pattern** — Agent-maintained knowledge: write-time size discipline, staged/promoted writes, and "continuous maintenance as the designed behavior" — the agent, not the human, carries the wiki burden.
- **#pattern** — Structured articles with links, provenance, and freshness, so agents route to the article the task needs rather than pulling whatever fits a memory cap.
- **#concept** — Memory-as-artifacts vs. memory-as-preferences: OzBrain's framing of what belongs in a brain (the work itself) against what platform memory stores (scraps and summaries).

---

## Critical Analysis

**What it gets right.** The context-ferrying diagnosis is real and under-served by the open-source memory systems catalogued in [[Agent Memory and Context]]. Those systems are overwhelmingly *single-tool* and *local*: [[napkin]], [[Claude-Mem]], [[Memento]], and [[Rowboat]] all assume the agent and the human live in one place. OzBrain is one of the few products that treats *cross-platform* as the primary problem rather than an afterthought — the same portability argument [[The Knowledge Chipper]] makes, productized. The "agents maintain it, humans don't" framing is also genuinely interesting: it takes the write-path insight from [[Context Graphs]] ("capture is a side effect of doing the work") and makes it the business model, sidestepping the wiki-death spiral that [[Memory Is a Mistake]] correctly identifies as the killer of human knowledge bases.

**What's thin.** This is a marketing page, and it shows. The "structured articles with links, provenance, and freshness" is described but never demonstrated — we don't see what a routing decision looks like, how contradictions between two agents' writes are resolved, or how provenance is checked against what the brain already holds ("every write is staged, routed, and checked" is asserted, not shown). The FAQ's comparison pages (vs. Mem0, vs. ChatGPT Memory, vs. Obsidian, vs. Claude Projects) are referenced but not included in the fetch, so the actual mechanisms stay opaque. The unnamed "OpenAI co-founder · Anthropic R&D" attribution for the load-bearing quote is a red flag — either it's a specific person being coyly cited, or it's borrowed authority.

**The same unresolved problems.** Every hard question from [[Memory Is a Mistake]] applies unchanged: retrieval policy (what gets pulled into *which* prompt), the decision swamp of contradictory half-truths, and — most acutely — persistent prompt injection into memory. OzBrain's answer to "who resolves contradictions" appears to be "write-time size discipline and staging," which bounds article size but not article *truth*. And the hosted architecture trades the local-first control that [[Memento]], [[Mnemo]], and [[Rowboat]] offer for convenience — encrypted-at-rest and markdown-export are good, but the whole point of a brain is that it becomes load-bearing, and a load-bearing brain at a third-party vendor is a different trust calculus than a git repo in your vault.

**Where it sits.** OzBrain is the commercial pole of the shared-memory spectrum this wiki has been tracking: at one end [[robot.wtf]] and [[Rowboat]] (git/file-based, human-visible, local), at the other OzBrain (hosted, connector-native, agent-maintained). It is closest in spirit to [[jibrain Knowledge Architecture]] and [[LLM Wiki]] — the "AI-maintained knowledge base" pattern — but with the friction removed in exchange for a subscription. The pricing curve (50 → 300 → 600 articles) quietly encodes the real product bet: that one brain per venture, kept current by agents, is worth $20–99/mo to someone who would otherwise be manually ferrying context.

---

*Sources: [[raw/ozbrain-com]], [[summary/ozbrain-com]]*
*Last updated: 2026-08-22*
