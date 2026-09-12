---
url: https://news.ycombinator.com/item?id=47368874
title: "How do you capture WHY engineering decisions were made, not just what?"
author: zain__t
date_fetched: 2026-05-22
date_published: 2026-03-14
topics:
  - software-engineering-craft
---

Original post by zain__t on Hacker News. 41 points, 64 comments. [flagged].

## Original Post

The author describes onboarding a senior engineer who spent three weeks doing "code archaeology" to understand *why* the codebase was structured a certain way. The engineer could quickly grasp *what* the code did, but the reasoning behind specific choices — like using Redis over in-memory caching, GraphQL for one service but REST elsewhere, or a peculiar auth exception for enterprise users — was buried in undocumented PRs, old Slack threads, and in the heads of departed engineers.

The author tried ADRs (lasted 6 weeks), PR description templates (ignored within a month), and a Notion architecture doc (unmaintained for 14 months). The core frustration: "Every solution requires someone to manually write something. Nobody does."

The post asks three questions:
1. Does anyone have a system that works long-term?
2. Has anyone automated any part of this?
3. Is everyone quietly suffering through this on every new hire?

## Selected Comments

### lowenbjer
Key insight: "documentation survives when it lives next to the code." File-level headers, good READMEs, and per-folder documentation work for humans, LLMs, and search. ADRs and Confluence die because they're separate from the code. Suggested an LLM-as-judge git hook that checks PRs for consistency with existing docs and blocks merges if updates are needed. Admits discipline is unavoidable but friction must be minimized.

Follow-up: when asked if the git hook gets bypassed, said to check back in two years but that "i think LLMs will enable us to handle information in a much better and smoother way."

### hiyer
Noted that 15+ years ago, decision recording in code comments was standard. Things moved to Jira and Confluence, making them undiscoverable. AI search tools have improved things, but the preference remains for documentation in the code.

### hermitcrab
Worked on "design rationale" recording ~25 years ago, noting it's a major problem especially for long-lived artifacts like nuclear reactors. Identified three reasons people don't document *why*:
- It may feel like reducing career security
- It could open them up to potential prosecution
- It takes significant time

Countered passive capture idea: a "busy engineer trying to hit a deadline is just going to do the easiest thing" and tacit knowledge is hard to capture automatically.

### physicles
Argued that for the first time, good docs pay dividends because LLMs love reading them and keeping them current. Maintains a personal "rude Q&A document" answering big questions (like "Why Kafka?"). Believes ADRs are point-in-time RFCs, not documents to maintain. "PRs and commits are a pretty terrible place to bury this stuff" for anything beyond a single commit's scope. Noted new hires are in the best position to update docs because they remember what it's like not to know.

### nonameiguess
Called ADRs "the only way I've ever seen it done well" for sufficiently large projects. Cited the IETF RFC model. Described the best code archaeology experience from the full Atlassian suite with all integrations turned on — commits and PRs had Jira ticket numbers, which linked to stories, which linked to ADRs with peer review. Noted this came with waterfall tradeoffs: slow delivery but unmatched non-functional requirements.

### hysan
Writes lengthy PR descriptions despite feedback that nobody reads them. Does it primarily for personal recall: "I do it mostly for me because I find it invaluable as I prefer writing shit down instead of relying on my flaky memory."

### wesselbindt
Blunt answer to "Why Redis over in-memory cache?": Sometimes "the dev had a hammer and the codebase was starting to look an awful lot like a nail."

### rich_sasha
Argued this problem is "sort of unavoidable" at smaller scales. Many decisions happen for no good reason, by accident, or for outdated reasons. Non-solutions offered: document high-level principles, keep people around, write wiki pages as breadcrumbs without expecting perfect currency.

### soniclettuce
Found ADRs didn't work great in practice due to discoverability and boundary issues. Noted ADRs are point-in-time records — you don't update them, you write new ones. One workplace had new hires document all their onboarding questions/answers, which quickly fixed incorrect docs.

### al_borland
Uses code comments for all "why" information, with templates like: "This previously used ${old-solution}, but has moved to ${new-solution} because ${reason}" or "This is ugly and doesn't make sense, but ${clean-logical-way} doesn't work due to ${reason}." Emphasized that "the code is the only thing I can trust to be there" through platform migrations and tooling changes.

### iSnow
Built an agentic framework that distills ADRs from Teams meeting transcriptions where everyone discusses freely. "Works surprisingly well" without requiring manual effort. Built on company time in ~2 weeks with Claude.

### lwhsiao
Hot take: "hire people that value writing. Create a culture around that." Cited Oxide Computer Company's rigorous RFD (Request for Discussion) process.

### sph
Directly asked OP: "Cut to the chase, what are you selling?"

### durzo22
Called this an "LLM post."

### hammadfauz
Detailed workflow: file issues in a tracker, prefix every commit with the issue ID, one branch per issue, one PR per branch, don't squash merge. Creates traceable chain from git blame → commit message → issue → branch → PR comments.

### sdeframond
Referenced Chesterton's Fence — sometimes the way to understand a fence is to remove it and see what happens. But acknowledged this is dangerous: "it's a lot harder to notice when valid data DISAPPEARS." Emphasized putting comments next to weird code and writing tests.

### andrewf
Suggested the new hire should write down everything they learn, so the next person only needs to cover the intervening time. Suggested LLMs might excel at answering questions from an unorganized mass of existing artifacts without curation effort.

### gardenhedge
Argued the company is missing an architect role — someone who would know and have documented why Redis over in-memory cache or why GraphQL was used in one service.

### CGMthrowaway
Proposed making the ADR a PR requirement then automating extraction of decision info. OP responded they're already automating this.

### 4b11b4
Master's thesis involves scaffolding ADRs, drawing a line between required human input and what can be safely scaffolded. Noted that decisions at higher architectural levels implicitly touch many code locations, making them hard to capture in-line.

### pxue
Uses Briefhq — an MCP/CLI connected to Claude Code and Slack with GitHub integration for recording decisions as contextual info.

### moltar
Leaves "as many crumbs as I can" in PR descriptions that become commit messages, linking issues, Slack threads, articles, and docs while explaining reasoning.

### rustyzig
Argued the specific implementation questions shouldn't matter — what matters is that system properties are "accounted for and validated" in tests or the type system.

## Notable Exchange
Throughout the thread, zain__t is building a product that passively extracts decision rationale from PRs, Slack threads, and tickets, then auto-drafts documentation for one-click approval. Multiple commenters either called this out as a stealth product announcement (sph, durzo22) or expressed skepticism (hermitcrab's "busy engineer" point).
