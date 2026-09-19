# Claude Code from Source

The landing page for an unauthorized 18-chapter book reverse-engineering Claude Code's architecture from the TypeScript source maps Anthropic shipped inside its npm package — nearly 2,000 files read in full, analyzed and written by 36 AI agents in about six hours. The page advertises the book's six architecture pillars (async-generator agent loop, 14-step tool pipeline, prompt-cache-sharing multi-agent orchestration, file-based LLM-recall memory, startup performance engineering, two-phase skill loading with hardened hooks) and promises five transferable "Apply This" patterns per chapter.

---

## What the page advertises

- **Agent loop**: one async generator drives everything — streaming model output, executing tools, recovering from errors, compressing context across four layers.
- **Tool execution**: a 14-step pipeline from model request to tool result, with permission resolution, speculative execution, and concurrent batching classified by safety.
- **Multi-agent**: sub-agents share prompt-cache prefixes for a claimed 95% cost cut; fork agents, coordinator mode, swarm teams with mailbox messaging.
- **Memory**: file-based, no database; four memory types, staleness warnings, and a Sonnet side-query claimed to beat embedding search.
- **Performance**: 240ms startup via parallel I/O; slot reservation; bitmap pre-filters for fuzzy search.
- **Extensibility and security**: two-phase skill loading (metadata at startup, content on demand); 27 lifecycle hooks with config snapshots frozen at startup to prevent injection.

## Key quotes

> "When Claude Code shipped on npm, the source maps came with it. We read every file."

The provenance claim, stated without apology. The book exists because of a release-engineering accident — `sourcesContent` fields left in shipped `.js.map` files. Compare [[Reverse Engineering Claude Code's Antspace]], where the leak was an unstripped Go binary inside the sandbox: this is now a pattern, Anthropic repeatedly shipping its internals to anyone who looks.

> "How sub-agents share prompt cache prefixes to cut costs by 95%."

The most economically interesting claim on the page. It says cost is an architecture constraint, not an afterthought: the multi-agent topology is shaped around where the cache boundary falls. If the 95% figure survives contact with the book's chapters, it belongs in any serious discussion of agent orchestration economics.

> "Four memory types, staleness warnings, and a Sonnet side-query that beats embedding search."

A direct challenge to the vector-database default. This nuances [[Memory Is a Mistake]], which concluded retrieval policy — not storage — is the hard problem: here the proposed retrieval policy is *an LLM call*, paying tokens instead of maintaining embeddings and hoping they stay aligned with files that keep changing.

> "27 lifecycle hooks with config snapshots frozen at startup to prevent injection."

Hooks are the most powerful and most dangerous Claude Code surface; freezing their configuration at startup is a specific, checkable hardening claim. See [[Claude Code Skills System]] for the other half of the extensibility story — the two-phase loading described here is progressive disclosure applied to the skill system itself.

## The provenance story is the story

Two facts elevate this above a typical vendor-teardown landing page. First, the source of truth is not interviews or guesses but the actual shipped TypeScript — this is archaeology, not journalism. Second, the book is an AI artifact about an AI artifact: 36 agents, four phases, six hours, then a final audit pass to ensure "no verbatim source code remained — every code block was rewritten as pseudocode." That audit pass is quietly the most revealing line on the page: generation was nearly free, so the bottleneck was verification and de-contamination — making sure the output didn't carry Anthropic's literal code out with it.

## Critical take

Treat every number here as marketing until the chapters back it up: 95%, 240ms, 27 hooks, 14 steps — this is a sales page for a book, not the book, and none of it is checkable from this page alone. The "purely educational" disclaimer, the pseudocode rewrite, and the parody "NO'REILLY" cover are a legal fig leaf over a contested act: extracting and publishing the architecture of proprietary software. The pseudocode rewrite also cuts both ways — it lowers copyright exposure while destroying verifiability, since you can no longer check a claim against the code it describes. The authors implicitly concede this tension by bragging about the audit that sanitized their own evidence. Still, the advertised claims cohere with what independent extraction efforts ([[Reverse Engineering Claude Code's Antspace]]) have found, and the pattern catalog — cache boundaries as architecture, LLM-as-retriever, startup-time-as-UX, frozen hook configs as injection defense — is exactly the checklist a production agent harness review should ask for. If the chapters hold up, this is the best documented single-agent architecture study in the wild; if they don't, it's a well-formatted hallucination with a parody cover. The wiki should treat it as a high-value, unverified secondary source and say so wherever it gets cited.

**Themes**: #tool #pattern #concept

## Related pages

- [[Reverse Engineering Claude Code's Antspace]] — a second extraction of Claude Code's internals (unstripped Go binary vs. this book's source maps); together they show Anthropic's shipped artifacts leaking architecture as a recurring pattern, and this book strengthens that note's method with a systematic rather than opportunistic example.
- [[Claude Code Skills System]] — the book's two-phase skill loading (metadata at startup, content on demand) is the same progressive-disclosure architecture that note documents, now claimed as a measured design decision with startup-cost motivation.
- [[Memory Is a Mistake]] — both sources argue memory architecture is really retrieval policy; this book's "Sonnet side-query beats embedding search" complicates that note's survey by proposing tokens-over-vectors as the answer.
- [[Agent Orchestration]] — the swarm-teams-with-mailbox and shared-prompt-cache-prefix claims feed directly into that topic's economics and topology threads: sub-agent design constrained by where the cache boundary falls.

---
*Sources: [[raw/claude-code-from-source-com]], [[summary/claude-code-from-source-com]]*
*Last updated: 2026-09-19*
