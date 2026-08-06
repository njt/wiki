# The Knowledge Chipper

A practitioner's diagnosis of the silent waste in AI-assisted development: agents do vast amounts of context-building work that vanishes when the session ends, leaving only a commit message and code behind. The problem isn't just inefficiency — it's a structural blind spot that makes code review nearly impossible and LLM portability a fantasy.

---

## Key Quotes

> "In my day to day development work, I find that my agents have to build up an incredible amount of knowledge about the problem I set them on. They scan files. They search API docs. They do a lot of work to get their context sufficiently full to be able to squirt out the relatively tiny number of final tokens that go into an actual code change."

The opening observation names the ratio that should alarm us: thousands of tokens burned on context-gathering, a tiny fraction on the actual output. The developer analogy — reading lots of code, building a detailed mental model, editing just the files you need — makes it concrete. The difference is that the developer *keeps* that mental model; the LLM doesn't.

> "I cannot estimate the millions of tokens that are burnt this way."

On two teammates using different LLMs against the same codebase, each rebuilding context from scratch. This isn't theoretical — it's happening right now in every org with mixed tooling. The waste is invisible because each individual session looks productive, and only the aggregate picture reveals the duplication.

> "The whole concept of 'LLM portability' sounded a bit high-minded to me, honestly. Like monomorphism. Or gluten intolerance."

The joke lands because it's true — LLM portability *did* sound like an abstract concern until it wasn't. The pivot comes with a story about a friend in Dubai describing AWS Bahrain being disrupted by war. Companies in the region "absolutely *cannot* let their AI use be run in another region because of… well… **war**." When a region goes dark, the ability to resume Claude's work on Codex (or vice versa) stops being academic and becomes existential.

> "Being a tech lead for a fast moving project was hard work *before* LLMs. Now, every PR is a freshly minted parcel of (un-|semi-)documented complexity."

The sharpest diagnosis in the piece. The junior dev vibe-codes a huge privacy change and goes home. The senior engineer who actually knows the privacy layer has to review it with zero of the context the junior's LLM built up. They can't resume a session they never started. Their only option: ask *their* LLM to rebuild all the same context. The circular waste is the point.

> "If I come back today to an area of the codebase that just had 250K tokens spent on it yesterday… am I really about to spend many more tens-of-thousands of tokens loading up the context window again?"

The closing question, left unanswered because there is no answer — this is just how things work today.

---

## Key Themes

- **#concept Context evaporation:** The LLM's "mental model" of a codebase — built through scanning files, searching docs, understanding architecture — is ephemeral. It dies with the session. The ratio of context-building tokens to output tokens is heavily skewed toward work that gets thrown away. This is the knowledge chipper: a machine that shreds understanding into commit messages.

- **#concept LLM portability:** Cross-model context transfer matters for practical reasons (regional outages, team tool diversity, model experimentation), not ideological ones. The Bahrain war story grounds this in reality: if your AWS region goes dark and your LLM vendor depends on it, you need to resume on a different model immediately. The article frames portability as disaster recovery, not idealism.

- **#pattern The reviewer's context gap:** When agents produce code, the only artifacts are the diff and the commit message — none of the reasoning, exploration, or false starts survive. A reviewer (human or LLM) must reconstruct all that context from scratch, burning tokens and time. This is the structural cause of the 441% review-time increase Osmani documents. Varda's moratorium on AI-written commit messages names the same problem from the artifact side; this article names it from the *process* side.

- **#concept The multi-model tax:** Every model switch on the same codebase incurs a full context rebuild. Teams using multiple LLMs (Codex for one teammate, Claude for another) pay this tax invisibly, per session, per developer, per day. The aggregate waste is "millions of tokens" and nobody's measuring it.

---

## Critical Analysis

**What's right:** The core observation — that LLM coding sessions are massively front-loaded with context-building that then disappears — is correct and under-discussed. The community has focused on *memory persistence* (saving what the agent learned) and *continuity* (resuming sessions), which address half the problem. What's missing from the discourse is the *asymmetry of waste*: the context-building work dwarfs the output, and both go away. This isn't a memory problem or a continuity problem — it's a *capital destruction* problem. You're paying for tokens, getting understanding, and then burning the understanding.

The LLM portability angle is genuinely novel in the practitioner discourse. Most discussions frame portability as a competitive concern (don't get locked into one vendor) or a philosophical one (models should be interchangeable). The Bahrain example reframes it as *infrastructure resilience* — the same reason you'd want your database to be portable across cloud providers. This is a more grounded and actionable argument than anything in the philosophical portability debate.

The code review connection is the piece's most important contribution. It connects two problems that are usually discussed separately: context loss between sessions, and the exploding code review bottleneck. The link is causal — the review bottleneck exists *because* context is lost. The junior dev's LLM built a detailed understanding of the privacy layer, made a change, and discarded the understanding. The senior reviewer must rebuild it. This is the mechanism behind every statistic about review times ballooning.

**What's missing:** The article is a diagnosis, not a prescription. It names the problem vividly but offers no mechanism for solving it — no proposed format for context artifacts, no workflow for capturing LLM reasoning, no tooling suggestion. This is fair for a blog post but leaves the reader in the same position the author describes: knowing something is broken without knowing how to fix it.

The piece also doesn't engage with existing work in this space. The "[[The Session You Cannot Take With You]]" article is referenced, but there's no mention of [[Coding Agents Continuity Not Memory]] (the continuity-vs-memory distinction directly addresses this problem), [[Agent Memory and Context]] (the hub page for exactly these concerns), or any of the concrete continuity implementations ([[napkin]], [[Claude-Mem]], [[AICTX]]). The author may not be aware of these, or they may believe they don't solve the specific problem being described.

The "millions of tokens" claim is evocative but unmeasured. The article doesn't attempt to quantify the actual waste — no back-of-the-envelope math, no session analysis. This weakens the argument for anyone who hasn't already felt the problem viscerally. A single measurement (e.g., "my last session: 87% context-gathering tokens, 13% output") would have made the case.

**The structural tension:** The article describes a problem that gets *worse* as models get better. When models could only produce small, simple changes, the context gap between author and reviewer was manageable. As models produce larger, more nuanced changes (referencing Philip's piece on AI outpacing human review capabilities), the gap widens. The very capability improvement that makes agents more useful also makes their output less reviewable. This is a real and troubling dynamic — and it's the same point [[The New Software Lifecycle]] makes about "AI turns implementation from writing into reviewing."

---

## Cross-References

- [[Agent Memory and Context]] — Hub page for context engineering. The knowledge chipper problem is context loss at a different level: not "how do I store what the agent learned" but "the agent learned a huge amount and none of it was captured."
- [[Coding Agents Continuity Not Memory]] — Santi's continuity-vs-memory distinction directly names this. The article is a practitioner's lived experience of exactly the cold-start problem continuity is supposed to solve.
- [[Agentic Code Review]] — Osmani's data on 861% churn and 441% longer reviews. This article provides the *mechanism* behind those numbers: reviewers have to rebuild context that was built and discarded.
- [[AI-Written Change Descriptions]] — Varda's moratorium because commit messages describe the obvious and omit the framing. This article extends the argument: not just commit messages, but the entire context behind a change is lost.
- [[The New Software Lifecycle]] — Osmani's map of uneven compression. The review bottleneck this article describes is the downstream consequence of upstream compression.
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — The argument that bigger context windows don't solve cold starts. This article is a field report confirming that claim.
- [[Context Engineering at the Frontier (Linus Lee)]] — Lee's composable retrieval pipelines. The knowledge chipper is the problem composability is meant to solve: make context reusable rather than disposable.
- [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — Brian Houck's synthesis: AI compressed upstream coding, everything downstream is breaking. The reviewer's context gap is one of the things that's breaking.
- [[Human-in-the-Loop is Tired]] — Laura Summers on the psychological cost. The senior engineer having to rebuild context they never had is this cost made concrete.

---
*Sources: [[raw/the-knowledge-chipper]], [[summary/the-knowledge-chipper]]*
*Last updated: 2026-08-06*
