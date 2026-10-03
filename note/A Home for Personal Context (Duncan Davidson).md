# A Home for Personal Context

Duncan Davidson argues that the model of you that each AI agent builds should live in a canonical, user-controlled repository that any agent can request permission to use — not in fragments locked inside each vendor's product. This page is his pointer post to the full O'Reilly Radar article, but the framing is complete enough to analyse on its own.

---

## What it argues

Every agent accumulates a private model of its user: Claude learns your prose, ChatGPT remembers your projects. Davidson doesn't object to the modelling — he objects to the *location*. The knowledge is stranded: switch products and you start over; run three agents and each rebuilds from scratch what the others already know.

The fix is architectural, not product-level: a **canonical, user-controlled repository of context**, with agents requesting *permission* to use it. He pairs it with his earlier thesis on personal websites as "canonical, public context" — the website is where you teach the world; the private richer context (preferences, projects, history) "has no home of its own" and lives in fragments inside whichever agent you've been using.

He acknowledges the existing folk pattern — pointing agents at a pile of Markdown files, with Karpathy's LLM Wiki as the exemplar — and reports that a year of working that way taught him five requirements a personal context system must satisfy. The closing question is the sharpest line in the piece: where should context live? "Not on which disk, but inside which trust boundary?"

---

## Key quotes

> "Every agent I use is building a model of me — and each one keeps it inside its vendor's walls. It doesn't need to be this way."

The framing is deliberately vendor-critical without being anti-agent: the problem is tenancy, not capability.

> "What if every person had a canonical, user-controlled repository of context that any agent could request permission to use?"

Note the *permission* language — this is a consent architecture, closer to an OAuth-scoped data store than to a sync folder. That's the part most Markdown-pile implementations quietly skip.

> "Where should that context live? Not on which disk, but inside which trust boundary?"

The right question. Local files aren't automatically safe (agents read them and exfiltrate via prompt injection); vendor clouds aren't automatically bad (they're just not portable). The unit of analysis is trust, not storage.

---

## Key themes

#concept #personal-agents #context-ownership #data-portability

## Critical analysis

The strength of the piece is the diagnosis: memory lock-in is real, growing, and almost never discussed as a *portability* problem. Every agent's memory feature (ChatGPT Memory, Claude's project knowledge, Claude Code's CLAUDE.md and Dreaming) is a walled garden by default, and Davidson's "three agents, three redundant models of me" framing makes the waste vivid. Coupling it to the personal-website argument gives the idea a nice public/private symmetry.

The weakness is what the pointer format hides: the "five things a personal context system has to get right" are behind the Radar paywall-link, and the hard problems live exactly there — access control granularity, schema evolution, how an agent *requests* context without the request itself leaking, and whether vendors have any incentive to read from a store they don't own. History (data portability regulations, Health Records, RCS) suggests incumbents adopt user-owned context stores only when forced. The Markdown-pile movement is, in a sense, the market's grassroots answer that already works well enough to delay the standard.

Still, the trust-boundary question ages well regardless of whether the specific product materialises. And the "changing agents won't mean changing homes" aspiration is the correct north star: context is the user's capital, and it should appreciate across agent switches rather than reset to zero.

## Related pages

- [[LLM Wiki]] — Davidson explicitly builds on Karpathy's pattern; this piece is the step from "a wiki the LLM maintains" to "a wiki the *user owns* and any agent may borrow," adding the consent and portability layer the wiki pattern leaves implicit.
- [[Coding Agents Continuity Not Memory]] — Santi's "continuity, not bigger memory" thesis is the engineering-side counterpart: Davidson says the *home* of context is wrong; continuity-not-memory says the *primitive* is wrong. Both reject vendor-owned memory as the answer.
- [[Agent Memory]] — Angie Jones's seven-type taxonomy describes what memory an agent needs; Davidson's piece supplies the political answer to where that memory should be *held* — user-controlled, permission-gated — which vendor implementations like OAMP don't address.
- [[Guardian Angels]] — Gwern's digital-twin vision depends on exactly the accumulated personal model Davidson says is currently trapped per-vendor; a canonical context store is the substrate a guardian angel would need to survive product churn.

---
*Sources: [[raw/a-home-for-personal-context]], [[summary/a-home-for-personal-context]]*
*Last updated: 2026-10-03*
