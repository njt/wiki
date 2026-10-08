# Devin's Memory and Dreaming

Cognition ships memory for Devin: a personal Git-backed "Memory Drive" of markdown lesson notes indexed by a `MEMORY.md` loaded at session start, plus "Dreaming" — a daily asynchronous consolidation session that deduplicates, prunes, and re-derives knowledge across all of a user's past sessions. The design is notable for what it borrows: code-navigation tools as the memory retrieval mechanism, git merge semantics as multi-session concurrency control, and a hard distinction between memory (accumulated context) and skills (packaged procedure). They also open-sourced the standard.

---

## What it does

- **Memory**: short notes with a link back to the session where the lesson was learned — "not summaries of sessions" but lessons from working with you. Personal per-user within an org, not shared instructions.
- **Memory Drive**: a persistent Git repo of markdown files, organized by repo/project/topic, with a `MEMORY.md` holding general preferences and an index. Sessions get only the index in context; the agent searches the rest "using the same tools it uses to navigate code."
- **Concurrency**: each session has its own checkout; commits are merged, stale writes are rejected by revision check, and conflicts are surfaced for resolution rather than silently overwritten.
- **Dreaming**: a daily background session that consolidates overlapping notes, removes transient details, harvests lessons that weren't captured live, and deletes stale records — while preserving source references and explicit preferences.

## Key quotes

> "Memories are not summaries of sessions. They are lessons Devin learned from working with you, written as short notes with a link back to the session where they were learned."

The distinction matters more than it sounds: session summaries rot into noise; lessons with provenance links stay auditable. Keeping the link back to the source session is the same move this wiki makes with raw/→note provenance.

> "Devin can search and read relevant notes using the same tools it uses to navigate code, without loading the entire memory archive into its prompt."

Index-first retrieval instead of everything-in-context — the standard answer to the context-limit pressure this wiki keeps circling.

> "Skills capture a repeatable workflow; memory captures what Devin learns while working with you... skills are deliberately packaged for reuse, while memory is accumulated through sessions and revisited through dreaming."

The first mainstream vendor to give the memory/skill split a clean lifecycle statement: deliberate packaging versus accretion plus offline curation.

## Themes

#concept #tool #pattern

- **Git as memory substrate** — merge semantics, revision checks, conflict surfacing: the boring, battle-tested answer to parallel sessions writing one memory store. Elegantly, the agent already knows git.
- **Consolidation as a scheduled job** — "Dreaming" is offline memory compaction with global context, the cron-shaped cousin of compaction-under-pressure.
- **Memory ≠ skills** — a vocabulary the field needed; most confusion in agent memory discussions is these two things blurred.
- **Open-sourcing the standard** — Cognition wants memory schema convergence the way MCP wanted tool-call convergence.

## Opinion

This is a sharp, restrained design — probably the best articulation yet of the "memory as a curated git repo of lessons, not a transcript" school, and the Dreaming frame is genuinely good naming for offline consolidation. Two caveats. First, the blog is marketing-thin on failure modes: nothing on bad lessons hardening into wrong lessons through consolidation (dreaming can amplify a mistake as easily as clean it), nothing on memory size growth, nothing on evals proving dreaming actually improves retrieval. Second, the memory-vs-skills dichotomy is cleaner in the announcement than in practice — a lesson learned twenty times is indistinguishable from a procedure, and the lifecycle boundary will blur in exactly the cases that matter most. Against skeptical takes like [[Maybe Coding Agents Don't Need a Bigger Memory]] and [[Memory Is a Mistake]], Devin's answer is: don't store more, store less but revisit it — which is a real position, and the open-sourced standard will let us test it.

## Related

- [[Memory Mechanism]] — the survey of agent memory architectures; Devin's Memory Drive is a concrete production instance of the index-plus-retrieval pattern it taxonomizes, strengthening its claim that retrieval beats stuffing.
- [[Three Tier Memory]] — the tiered session/medium/long design; Devin collapses the tiers into one git repo with dreaming as the compaction layer, a simplification that nuances the tier thesis.
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — the skeptical counterpoint; Devin complicates it by agreeing memory isn't about volume, but betting consolidation plus personalization is still worth the machinery.
- [[Memory Is a Mistake]] — the strongest anti-memory position; Devin's "preserve explicit preferences, prune the rest" is a partial concession to exactly the pollution risk that page raises.

---
*Sources: [[raw/memory-and-dreaming]], [[summary/memory-and-dreaming]]*
*Last updated: 2026-10-08*
