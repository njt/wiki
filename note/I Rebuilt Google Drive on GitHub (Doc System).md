# I Rebuilt Google Drive on GitHub (Doc System)

After a year of fighting the friction between Claude Code and Google Drive, the author gave up on integration and rebuilt the product instead: a "Doc System" where GitHub is the storage layer, every document is Markdown/HTML in git, an automated publisher commits to a protected `main`, and an internal web app (IntensityOS) re-creates the Drive experience — folders, tags, sharing, comments, suggestions — on top. The real point is architectural: agents work natively in git and Markdown, so the storage layer should be shaped for them and the human UI layered on top, not the reverse.

---

## What the system is

- **Drafts**: a private `docs-drafts` git repo on the laptop; every Claude turn that touches a file auto-commits it, so saving and versioning disappear as concerns.
- **Publishing**: a block list (personal, health, people documents) gets a never-publish marker at creation; the publisher refuses to touch them, runs gitleaks, then makes one atomic commit to `main` — no branches, no pull requests. A webhook lands the doc in the IntensityOS catalog within a minute.
- **The human layer**: Drive-like folders, auto-tagging by an AI job, sharing via link or Google contacts, draft/final/archived stages, version history, a private area, comments and suggestions where an accepted suggestion becomes a git commit. Search is full-text plus vector embeddings with an Ask box answering with citations.
- **The agent layer**: "Fluffy", his laptop agent, has its own identity in IntensityOS — it searches, reads comments, suggests edits, replies, closes threads, and publishes. A `/co-pushdoc revise <doc>` command handles all open comment threads: makes the clear changes, asks about ambiguous ones, republishes, and closes each thread, with the record showing *who* closed it — agent or human.

## Key quotes

> "The Docs API hands back a document as a tree of elements, and edits are offsets into that tree. Claude can do it, but it fights the whole way."

The diagnosis of why agent–Drive integration failed: the API surface is shaped for GUI tooling, not for a model that thinks in plain text. This is the whole post in one sentence — the failure is architectural, not a prompting problem.

> "Drive works fine as an output. It just doesn't work as the place where the work happens."

The inversion that makes the design click: legacy SaaS doesn't need to be replaced, it needs to be demoted from system of record to an export target. Companies still wanting Drive get each doc as PDF and Google Doc.

> "Claude knows `git` and `gh` cold, so it doesn't need a special API to read a document or see what changed. The web app is for the humans."

The most transferable idea in the piece: when your agent already has deep fluency in a substrate (git), bolting a bespoke API on top is negative value. Choose storage formats the agent speaks natively.

> "The code is the easy part. The problem is me."

On not open-sourcing it: he knows what maintenance costs — issues, PRs, security reports, support — and won't pay it. An unusually honest take; most builders rationalize past this.

## Key themes

#concept #tool #pattern #personal-agents

## Analysis

The strongest claim here is that format choice *is* agent architecture. Markdown being "the native language of every LLM" means the conversion tax Google Docs imposes isn't incidental — it's the cost of a data model (a tree of elements, offsets into it) designed for humans clicking. The author didn't make Claude better at Drive; he moved the work to where Claude already excels. That's a design principle worth generalizing: **for agent-native systems, pick the substrate the model is pretrained on, then wrap the human UI around it.** Git doubles as versioning, access control, and the audit trail — with a protected branch giving the same immutability Drive approximates with version history, but verifiable.

The agent-identity detail deserves more attention than the post gives it. Fluffy closing comment threads under its own recorded identity, with the thread showing whether an agent or a human closed it, is a small but real governance pattern: accountability in the collaboration log, not in a separate audit system. It also sidesteps the fake-persona problem — the agent is an explicit, named participant rather than pretending to be the author.

The open-source ending is quietly the most instructive part. The wiki's own [[The Golden Age of Open Source Applications]] argues AI makes open source cheap to build; this post supplies the counterweight — *maintaining* remains expensive in a purely human way, and that cost is now the bottleneck, not the code. Someone will eventually productize the maintenance layer and unlock a shelf of exactly these half-released systems.

One thing the post underplays: this is a solo-built, company-internal system with an auto-publish pipeline straight to `main` and no review gate. It works because the author is the only author and gitleaks catches secrets. Hand the same pipeline to a team of humans *and* an agent that can publish, and the missing PR step becomes the threat model — the never-publish block list is a good start, but it's an allowlist-of-one story today.

## Related pages

- [[Wuphf — Karpathy-Style Agent Wiki]] — the same underlying bet (Markdown + git as the agent-native substrate, no conversion layer) applied to knowledge management rather than documents; this post strengthens it by showing the same pattern extended to a full collaboration workflow.
- [[The Golden Age of Open Source Applications]] — complicates its "AI makes open source cheap" thesis: here the build was cheap but the maintainer's obligation is the stated blocker to release.
- [[A Home for Personal Context (Duncan Davidson)]] — shares the conviction that personal data should live in a user-controlled, agent-reachable repository rather than inside a vendor's walls; this post is a working implementation of that for documents.
- [[Keep AI Out of Your (Obsidian) Vault]] — the opposite pole on the same axis: mehdio guards a private vault against AI contamination, while this author makes git-native Markdown the agent's primary workplace.

---
*Sources: [[raw/i-rebuilt-google-drive-on-github]], [[summary/i-rebuilt-google-drive-on-github]]*
*Last updated: 2026-11-17*
