# If AI Is Doing the Investigation, Version the Investigation

Mark Fletcher's argument that AI coding sessions are investigation records and should be versioned alongside the code they produce. His solution: "Cases" -- per-task directories committed in the repo containing human summaries, Claude transcripts, and distributed traces. The diff tells you what changed; the Case tells you why.

---

## Key Quotes

> "I was trying to reason through it with only half my brain."

The problem statement in one sentence. The diff is half the brain. The investigation -- the dead ends, the design tradeoffs, the constraints -- is the other half. Currently that half evaporates when the terminal tab closes.

> "I commit bug fixes, but all the knowledge around finding and fixing the bugs is lost."

This isn't just about bugs. It's about the design reasoning for features, the alternatives considered and rejected, the constraints that shaped the approach. All invisible in a diff.

> "Most Claude sessions are throwaway, same as most terminal sessions. Cases are for the ones worth keeping. You don't write a postmortem for every alert; you write one when it matters."

The selectivity is important. Not every session needs a Case -- only the ones where the investigation itself is the valuable artifact. This is a signal-to-noise discipline, not a hoarding instinct.

> "Because they're in the repository, they're available to all developers. They show up in PRs alongside the diff, so reviewers can see not just what changed but why."

The killer feature. Cases aren't siloed in an external tool or a personal notebook -- they travel with the branch. No stale links, no out-of-sync wikis, no "where did that investigation go?"

> "Claude in Trellis has read-only access to production systems. It can query logs, run distributed traces, and read crash reports, but it can't restart services or modify anything."

This is the safety architecture for production investigation: read-only access with the transcript as audit trail. The transcript proves the AI didn't do anything it shouldn't have, because the transcript *is* the record of everything it did.

> "If AI is part of how you build software, its work shouldn't vanish when you close the tab."

The thesis, distilled. This will age well.

## Key Themes

#concept #case-management #transcripts #version-control #agent-workflow #audit-trail

## What It Gets Right

**The right granularity.** Cases are per-task, not per-session or per-commit. A single bug fix might span multiple sessions; a single session might touch multiple bugs. The Case is keyed to the *investigation*, not the container it happened in.

**Repository-native is the right home.** External tools (Notion, Linear, Google Docs) have a half-life. The repo is the one artifact that survives. Putting investigation records in the repo means they survive the same way code survives.

**"Half my brain" is a diagnosis, not just a complaint.** It names the cognitive debt problem at the individual developer level. [[Cognitive Debt]] describes the organizational version -- velocity exceeding comprehension. Fletcher describes the personal version: looking at your own code from last month and not remembering why you wrote it that way.

**Read-only production access with transcript-as-audit-trail.** This is the security architecture that makes production AI investigation safe. The transcript isn't just documentation -- it's proof. If the AI can't write, and everything it reads is logged, there's nothing to audit for side effects.

**PR review augmentation.** Cases shown alongside diffs in PRs mean reviewers get the *why* alongside the *what*. This is a concrete mechanism for closing the comprehension gap that [[Cognitive Debt]] identifies -- and it doesn't require the reviewer to be more experienced or to spend more time. It just requires the investigation to be legible.

## What's Missing

**Curating Cases takes discipline.** Fletcher acknowledges that most sessions are throwaway, but the boundary between "throwaway session" and "Case-worthy investigation" is blurry. The first time you need a Case and don't have one, you'll wish you'd been more aggressive about creating them. The first time you create Cases for everything, you'll drown in noise.

**Cases don't solve discovery.** The Case is next to the code it produced, which works when you're `git blame`-ing a specific line. But when you want to know "what have we learned about ICS calendar bugs across the entire codebase?", Cases scattered across the repo don't help. You'd need something like [[workgraph]] or a search index.

**The human summary bottleneck.** `notes.md` is a human-written summary, which means it's subject to the same "I'll do it later" failure mode as postmortems. Fletcher is a solo developer on Groups.io, so he can enforce his own discipline. On a team, the `notes.md` would need to be part of the Definition of Done.

**Re-hydration is underspecified.** The claim that Trellis can "re-hydrate" Claude Code sessions from Cases is tantalizing but thin on details. How much of the original context is recoverable? Does the re-hydrated session actually pick up where the old one left off, or is it more like giving Claude a summary and hoping for the best?

## Connections

- [[Cognitive Debt]] -- Cases are a direct antidote: version the investigation so comprehension doesn't evaporate
- [[workgraph]] -- Same philosophy: the work is the durable artifact. "Agents can come and go, the graph remains" maps directly to "Cases survive sessions"
- [[napkin]] -- Another per-repo, markdown-based knowledge preservation pattern. napkin is agent-facing (the agent writes its own mistakes); Cases are human-facing (the human summarizes the investigation)
- [[engineering-notebook]] -- Automatic session diary. Cases are the curated, high-signal subset that engineering-notebook would produce if it had taste
- [[claude-replay]] -- Makes transcripts human-readable; Cases put those transcripts where they're useful
- [[Agent Memory and Context]] -- Cases are a new entry in the memory taxonomy: investigation memory, co-located with the code it produced
- [[Planning With Files]] -- Cases are planning-with-files applied to bug investigation: notes.md as the plan, transcripts as the execution trace
- [[How Intercom Uses Claude Code]] -- They sync session transcripts to S3 and analyze with Haiku. Cases are the repo-native, lower-infrastructure version
- [[Slate]] -- Thread-and-episode architecture. Cases are episodes: self-contained investigation records that can be resumed
- [[Harness Engineering]] -- Feedforward vs. feedback. Cases provide feedforward to the next developer (or yourself six months later) who touches this code

---
*Sources: [[raw/if-ai-is-doing-the-investigation-version-the-investigation]]*
*Last updated: 2026-05-14*
