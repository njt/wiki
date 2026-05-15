# ProofEditor

Proof is a free, no-login collaborative document editor built by [[Every]] for the specific use case of agents and humans writing together. Agents suggest edits (comments, inline suggestions), humans review and accept or reject. Every character is provenance-tracked to its author.

---

## Key Quotes

> "Fast, free, and no login required."

Commentary: The zero-friction onboarding is the product strategy. Unlike [[Mist]] (which targets humans collaborating on Markdown) or Google Docs (which is structurally unable to treat agents as first-class participants), Proof is designed from the ground up for the human-agent dyad.

> "Every character tracks who wrote it."

Commentary: Provenance is Proof's killer feature. In a world where agents generate 90% of document content, knowing which human wrote which sentence becomes the scarce signal. This is the document-level equivalent of `git blame`, and it's structural (built into the CRDT) rather than bolted on.

> During setup, Proof asks: when should new docs be opened in Proof? Options: all markdown docs; only collaborative docs like plans/specs; only when explicitly asked.

Commentary: This triage prompt reveals the product's self-awareness. Proof knows it shouldn't own every document — only the ones where agent-human collaboration is the point. This is better segmentation than most agent tools manage.

## Key Themes

#tool #collaboration #agent-human-interface #provenance #agentic-coding

Proof occupies a specific intersection: it's for documents that are *produced by agents, consumed by humans*. That's the reverse of most agent interfaces (where humans type instructions, agents read them). In Proof, agents write the first draft and humans edit.

This inverts the typical workflow. In [[Agent Coding Workflow]], the human writes a spec and the agent implements. In Proof, the agent writes the document and the human reviews. Both patterns share the same architecture: one party creates, the other verifies.

The suggestion-mode UX — agents propose, humans accept/reject/reply — mirrors the approval workflow in [[weft]] (agents work, humans approve state mutations) and the progressive trust model in [[Experience Design for Agents]]. It's the right default: agents can't unilaterally edit shared documents, only propose changes.

The API is clean and well-documented. The `edit/v2` protocol with mutation tokens and idempotency keys suggests the team understands distributed systems — this is a CRDT-backed editor, not a simple text field with an API slapped on.

## Critical Analysis

**What's smart:** Proof understands that the bottleneck in agent-human collaboration isn't model capability — it's the interface. Agents can already write good documents. What's missing is a surface where humans can efficiently review, annotate, and approve agent-generated content. Proof fills that gap with a thoughtful permissions model (agents suggest, don't execute) and structural provenance.

**What's underrated:** The local macOS bridge (`localhost:9847`) is a genuinely good idea. It means Proof works as a native app with local file access when you want it, and as a web app when you're sharing with remote collaborators. Most tools pick one mode; Proof does both.

**What's missing:** No version history, no diff view between agent and human edits, no export to [[Specifications as the Product|structured spec formats]]. The document model is flat Markdown — which is simple but limits what agents can express. Compare with [[The Plan Is the Program]], where the plan artifact has rich structure. Proof's documents are prose-first, which is right for memos and briefs but weak for implementation plans that need checkboxes, status tracking, and structured metadata.

**The Every connection:** Proof is built by Every (every.to), a media company that also builds software. This is unusual — most media companies don't ship developer tools. But Every's writers are power users of AI-assisted writing, so Proof is likely scratching their own itch. The question is whether a media company's internal tool generalizes beyond their use case.

**Comparison to [[poietic]]:** Both care about legibility of human-machine collaboration, but from opposite directions. Poietic is heavy infrastructure (dependency graphs, Rust, MIT license, public-benefit corp). Proof is light interface (share a URL, start editing). Poietic asks "how do we coordinate complex work?" Proof asks "how do we write a document together?" Both are right that the answer involves making contribution visible.

**Bottom line:** ProofEditor is the first tool I've seen that treats agents as first-class document collaborators rather than bolting agent access onto a human-first editor. The provenance tracking, suggestion-mode permissions, and API-first design are the right primitives. Whether it becomes essential depends on whether agent-generated documents become a distinct category of artifact — if agents are writing your PRDs, strategy docs, and research briefs, you want a specialized review surface for that content. If agents are just another set of fingers on the keyboard, Google Docs is fine.

---
*Sources: [[raw/proofeditor]]*
*Last updated: 2026-05-14*
