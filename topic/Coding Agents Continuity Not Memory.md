# Coding Agents Continuity Not Memory

Santi (oldskultxo) argues that "bigger memory" is the wrong framing for coding agents. The real primitive is **continuity**: preserving the operational thread across session boundaries. Context is what's available now; continuity is what lets the next execution continue from what actually happened before. The repo itself — not a cloud DB or chat history — is the natural home for this state.

---

## Key Quotes

> "A coding agent can have a large context window and still lose the operational thread. It can have chat history and still fail to know what happened in the last run."

The opening diagnosis. This is the core of why "just make the context window bigger" is an insufficient answer — it's solving the wrong problem. Context is breadth; continuity is depth across time.

> "Once you say memory, the temptation is to build a bigger one. A bigger context window. A bigger note store. A bigger vector database. A bigger archive of everything the agent has ever seen, said, touched, generated, or vaguely implied. That sounds powerful, but it is also how you build a very expensive junk drawer."

The naming trap. Calling the problem "memory" frames the solution as accumulation, which is actively harmful — stale, unverified, contradictory information mixed with facts at equal weight. This rhymes with [[Agent Memory and Context]]'s observation that every team independently discovers "what to forget" is the hard problem.

> "The difference between a memory item and a continuity record is huge. One is a recollection. The other is a handoff."

Santi's strongest distinction. A memory item ("we probably fixed the parser") is vague and unversioned. A continuity record includes files touched, commands run, results observed, known failures, next actions, and evidence quality. This is the operationalization that [[State System]] also pursues with evidence-first commits.

> "A vector store may retrieve something related. But relation is not enough. The next session needs provenance."

Directly challenges the vector-DB-as-memory orthodoxy. Semantic similarity doesn't answer: was this observed? inferred? validated? contradicted? still fresh? This is the provenance problem that [[napkin]] and [[Claude-Mem]] both run into — accumulation without lifecycle becomes archaeology.

> "Chat history is often bound to one provider, one tool, one account, one session, or one UI. But repositories outlive chats. The state that matters should live with the project."

The repo-as-boundary thesis. Contrast with [[Agentcookie]] which solves the cross-machine session problem differently (sync browser state, not project state). Both agree that continuity shouldn't be trapped in one session, but Santi argues the repo is the right substrate.

> "The loop turned out to be more important than any individual memory feature. Because without a lifecycle, memory is passive. It waits. It accumulates… It becomes archaeology. With a lifecycle, memory becomes part of execution."

**resume → work → finalize → resume.** This loop is the architectural discovery. It transforms memory from a passive store into an active participant in the execution cycle. Compare with [[Slate]]'s thread-and-episode architecture, which also separates active work (threads) from compressed results (episodes).

> "The continuity layer should minimize useless rediscovery. That means the best resume payload is not the largest one. It is the one that gives the agent enough operational grounding to avoid starting cold."

Counterintuitive and important: better continuity means *less* loaded, not more. The goal is the right door, not the entire memory palace. This inverts the instinct behind every "let's stuff more context in the prompt" approach.

> "Failure memory should not make the agent afraid of the repo. It should make it less naive instead."

Remembering failures is more valuable than remembering successes — but the framing matters. Not "never do this" but "this failed before, treat as context not truth."

> "If the guardrail becomes louder than the work, the system has failed."

On guardrails: they should fire at boundaries (first edit, risky command, scope change) and mostly say "allow." The design constraint is that continuity infrastructure must not become more expensive than the orientation cost it replaces.

> "A system that remembers everything will eventually force the agent to rediscover what matters inside the memory itself. That is just moving the cold start to another folder."

The pruning paradox. Accumulation-oriented systems just relocate the cold-start problem. Continuity-oriented systems force the hard question: what deserves to survive?

---

## Key Themes

- **#concept — Continuity vs. Memory**: The central reframe. Memory is passive accumulation; continuity is active handoff with lifecycle, provenance, and pruning. They're different categories that happen to share the word "memory."

- **#pattern — Resume-Work-Finalize Loop**: The three-phase lifecycle that turns memory from archaeology into execution infrastructure. Each session resumes from bounded state, does work, and records evidence for the next session.

- **#pattern — Evidence-Weighted Continuity**: Every fact carries a provenance tag (runtime-observed, agent-claimed, validation-supported, user-corrected, later-contradicted). The goal shifts from "believe memory" to "use it with the right weight."

- **#pattern — Repo-Local State**: Continuity lives as inspectable artifacts in the repository, not in a cloud service or provider-specific black box. This makes it reviewable, cleanable, and portable across tools.

- **#tool — AICTX**: The open-source repo-local continuity runtime Santi built to explore these ideas. CLI + MCP interfaces, failure memory, work state, execution contracts, guardrails.

- **#concept — The Pruning Imperative**: The hard part isn't capturing state — it's deciding what deserves to survive. Systems that optimize for accumulation eventually force the agent to rediscover what matters inside the memory itself.

---

## Critical Analysis

This is one of the more useful frames I've read on the agent memory problem, precisely because it's *not* a taxonomy paper or a product pitch. It's a practitioner's retrospective on building the wrong thing and discovering what actually mattered.

**What's genuinely new here**: The continuity-vs-memory distinction. Most writing on agent memory lumps everything together — context windows, vector stores, chat history, instruction files, note-taking. Santi's argument that these are different categories solving different problems is correct and clarifying. The resume-work-finalize lifecycle is also a concrete pattern that anyone can adopt today with markdown files, no tooling required.

**What's underdeveloped**: The pruning problem gets exactly one section near the end, but it's arguably the hardest part of the whole system. "Deciding what deserves to survive" is a judgment call that itself requires context — there's a recursion problem here. How does the continuity layer know what's stale without loading everything to check? The article acknowledges this is hard but doesn't offer a mechanism beyond "keep asking."

**The AICTX gap**: The article references AICTX as the implementation but doesn't link to it directly or describe what it actually does beyond the conceptual architecture. The "practical architecture" diagram is clear, but there's no sense of whether this is 200 lines of bash or 10,000 lines of Rust. For a piece arguing that implementations should be boring and inspectable, more implementation detail would strengthen the argument.

**What this gets right that others miss**: The economics section ("1-2 prompts probably not worth it") is refreshingly honest. Most tool authors pitch their solution as universally necessary. Santi draws a value curve and admits continuity has overhead that isn't justified for short tasks. This is the kind of engineering judgment that's missing from most agent-tooling discussions.

**Tension with existing approaches**: The repo-local philosophy is in productive tension with [[Agentcookie]]'s cross-machine session sync and [[mira-OSS]]'s narrative memory approach. Santi would argue those solve different problems, and he's right — but a production system probably needs all three: project continuity (AICTX), session portability (Agentcookie), and long-term learning (mira-OSS). The trick is keeping them from conflicting.

**The unstated assumption**: This entire approach assumes the coding agent is disciplined enough to run the lifecycle. A lazy finalize is worse than no finalize — it writes stale state that the next session trusts. The system's reliability depends on the agent's reliability at the recording step, which is the step agents are worst at (they're optimized for generation, not bookkeeping).

**The four-layer model as complement.** Codez's [[Giving Claude Agent Memory in 12 Steps]] provides the other half of the picture: a four-layer implementation ladder (Chat Memory → Projects → CLAUDE.md → Dreaming) that's about what the agent *knows*, while Santi's continuity is about what the agent *is doing*. They don't compete — an agent with continuity but no memory still has to re-derive preferences; an agent with memory but no continuity still loses its operational thread between sessions. The engineering is in the integration.

---

## Related Pages

- [[The Knowledge Chipper]] — A practitioner's field report of exactly the problem Santi diagnoses: an agent spends 250K tokens building understanding, the session ends, and the next session starts cold. The continuity-vs-memory distinction isn't theoretical — this is the daily experience it describes.
- [[Agent Memory and Context]] — Hub page. Santi's argument is a direct challenge to the "more memory" framing that dominates this space.
- [[Slate]] — Thread-and-episode architecture that separates active work from compressed results. The closest existing pattern to resume-work-finalize.
- [[State System]] — Evidence-first commits and deterministic replay. Shares the provenance obsession.
- [[napkin]] — The minimal version of this idea: a markdown file per repo. Proves you can get 80% of the value with zero infrastructure.
- [[Planning With Files]] — Context window = RAM, filesystem = disk. The conceptual precursor.
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management as the real engineering challenge. Independent discovery of the same problem.
- [[Writing a Good CLAUDE.md]] — Repo instructions as static context. Santi's point: instructions are necessary but insufficient without dynamic work state.
- [[Guardrails and Feedback Loops]] — Guardrails at boundaries. Santi's "allow / caution / re-ground / block" is a concrete implementation.
- [[Agentcookie]] — Cross-machine session continuity via synced browser state. Complements rather than competes.
- [[Claude-Mem]] — Hook-based memory with lifecycle. The closest existing tool to Santi's vision.
- [[StrongDM Factory Techniques]] — Filesystem-as-memory pattern. Repo-local artifacts as agent substrate.
- [[Agent-Native Architectures (Every)]] — Files as universal interface. The repo-as-boundary idea taken to its logical conclusion.
- [[Components of a Coding Agent]] — Harness matters more than model. Continuity is a harness concern.

---

*Sources: [[summary/coding-agents-continuity-not-memory]]*
*Last updated: 2026-06-09*
