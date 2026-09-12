# Organizing Claude Code for Product Work

A product manager's account of converting Claude Code from a chat-style tool into a persistent filing system: a three-folder workspace keyed to rate of change (`context/`, `projects/`, `operations/`), a self-maintained `CLAUDE.md`, and a set of skills that turn recurring work into one-time setup. The thesis is that past the basics, results stop depending on how well you prompt and start depending on how well you file — the author arrived at the system by getting it wrong first, then packaged it as a public `claude-code-pm-starter` template.

---

## Key Quotes

> "Past the basics, your results in Claude Code stop depending on how well you prompt and start depending on how well you file. A good prompt improves one session; a good file improves every session after it."

The article's load-bearing claim, and the cleanest articulation of the "workflow over prompts" school that runs through [[Tuning Claude Code Into a Better Engineering Partner]]. It relocates the bottleneck from prompt craft to information architecture: the thing that compounds is not what you say but where it lands.

> "Chat is a great place to think and a terrible place to accumulate."

The diagnosis that motivates the switch to Claude Code. Every chat session starts from zero — the uploaded strategy doc is gone, the explained context lives in an unfindable thread, the liked output is trapped in a conversation. Claude Code's difference is a folder home: context in files that persist, recurring work in skills that run on command, outputs as documents you keep. This is the same cold-start problem [[Maybe Coding Agents Don't Need a Bigger Memory]] and [[Coding Agents Continuity Not Memory]] name from the engineering side, reframed for a non-technical user.

> "File things by their speed, and stale can never masquerade as fresh. File them by topic, and old truth sits next to current truth, looking identical."

The genuinely novel contribution. The organizing question isn't "what topic?" but "how fast does this change?" — positioning barely moves, a status summary lasts a week, a ticket queue changes by the hour. The author explicitly positions this against PARA: Tiago Forte sorts by actionability because a human's failure mode is losing track of what to act on; **this system sorts by rate of change because an LLM's failure mode is stale context confidently reused.** That last clause is the sharpest insight in the piece.

> "The test that settled it: nouns go in files, verbs go in skills."

The know/do split, arrived at independently but converging exactly on [[Steering Claude Code]]'s "CLAUDE.md-as-facts, skills-as-procedures" taxonomy and Diátaxis's decade-old documentation split. Facts (product facts, user segments, terminology) are nouns and live in `context/`; procedures (draft, review, synthesize) are verbs and become skills. The author even names the two failure modes the split catches: a "skill" that's really a fact bucket, and a documented-but-not-executable procedure that gets re-explained anyway.

> "If you've said it twice, it belongs in a file."

The one-line memory discipline the whole system reduces to. A correction made in chat is perishable; the compounding return comes from making corrections permanent. The mechanics are concrete: three lines in `context/preferences.md` about how you write, and a `/file-feedback` skill that routes each correction — one-off slip, missing fact, wrong process, or taste — with a "why" line and a "how to apply" line so future sessions judge edge cases instead of pattern-matching blindly. It rhymes with the Two Corrections Rule in [[Tuning Claude Code Into a Better Engineering Partner]] and Anthropic's own memory docs.

> "Stale context is worse than missing context."

The pruning argument, made in the closing rhythm section. An expired competitor note doesn't look stale to Claude; it gets woven into new work with full confidence. The practice — date-stamp what's dated, archive closed projects, delete anything you wouldn't want quoted back — is the maintenance habit most filing systems omit.

## Key Themes

#claude-code #skills #CLAUDE.md #context-engineering #filing #product-management #pattern #tool

## Critical Analysis

**The rate-of-change principle is the real contribution, and it's under-titled.** Every personal-knowledge-management system sorts by topic or actionability; almost none sort by decay rate. For an LLM the failure mode genuinely is different from a human's — a human re-reads an old note with skepticism, an LLM reuses it with confidence. Making "how fast does this change" the primary axis is a correction to PARA worth stealing even if you ignore the rest of the template. It also gives the three-drawer architecture (context / projects / operations) a *reason* rather than a vibe.

**The know/do split is correct but not novel — and that's fine.** The author presents "nouns in files, verbs in skills" as a personal discovery, but it's precisely [[Steering Claude Code]]'s CLAUDE.md-as-facts / skills-as-procedures distinction, which itself traces to Diátaxis. The value isn't the novelty; it's that a non-technical PM converged on Anthropic's official taxonomy independently, which is decent evidence the taxonomy is load-bearing rather than arbitrary.

**The audience is both the strength and the ceiling.** Most Claude Code field guides assume an engineer ([[Claude Code Mastery]], [[Tuning Claude Code Into a Better Engineering Partner]]). This one spells out what an IDE is and why a terminal alone is "a bad home for a product manager." That on-ramp is the point — but it means the article skips the failure modes the engineer-facing guides foreground: attention budgets, compaction dropping skills, cost, hook-based enforcement. The under-200-lines CLAUDE.md claim is asserted from Anthropic's context-engineering post, not measured the way [[Tuning Claude Code Into a Better Engineering Partner]] measures its 40K→6K shrink.

**The single-player blind spot.** The workspace is shareable — "push it to a private repository and a teammate can clone it and inherit your head start" — but the author stops there. [[Team-Wide Agentic Harness]] is the natural next page: what happens when two people's `context/` files drift, who reviews the skills, and which conventions survive contact with someone else. The article gestures at multiplayer ("the line I'd hand your team") without engaging its collision problem.

**What's missing is the cost side.** The system compounds context, but context costs tokens, and the article never mentions money. [[Managing AI Coding Costs at Scale]] and [[What a User Story Actually Costs in a Dark Code Factory]] are the counterweight: a filing system that grows without pruning is a bill that grows too. The author's "prune every dozen sessions" is the right instinct but leaves the economic pressure unnamed.

## Connections

- [[Steering Claude Code]] — the official taxonomy this article converges on: facts vs. procedures, the 200-line CLAUDE.md budget, and why the map stays small
- [[Tuning Claude Code Into a Better Engineering Partner]] — the closest sibling: "workflow over prompts" plus the Two Corrections Rule, which the "file every correction once" practice restates
- [[Claude Code Mastery]] — CLAUDE.md as "compounding infrastructure" and skills as reusable expertise, from the engineer's side
- [[Team-Wide Agentic Harness]] — the multiplayer question this article raises but doesn't answer: the harness as reviewed, version-controlled team infrastructure
- [[Agent Memory and Context]] — the synthesis hub for the persistence problem this system is a worked answer to
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — the cold-start diagnosis ("context ≠ continuity") behind why a folder beats a chat thread

---

*Sources: [[raw/how-to-organize-claude-code-for-product]], [[summary/how-to-organize-claude-code-for-product]]*
*Last updated: 2026-09-04*
