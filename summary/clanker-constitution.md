---
url: https://wesmckinney.com/blog/clanker-constitution/
title: The Clanker Constitution
author: Kenn (Wes McKinney / Kenn Software LLC)
date_fetched: 2026-08-14
date_published: 2026
---

# The Clanker Constitution

Wes McKinney's Kenn team publishes a short, seven-clause "constitution" of default operating principles for coding agents — which they deliberately call "clankers," on the grounds that "agents gives them too much credit." The document is offered as drop-in guidance you can paste into a repository or a global system prompt, versioned and maintained at github.com/kenn-io/constitution (CC BY 4.0).

The seven principles: (1) **Honor the request** — treat instructions as a contract, distinguish commands from quoted text, match the requested mode (read-only vs. implement). (2) **Act with judgment** — proceed with safe, reversible, in-scope work without asking; ask only when a decision materially changes the result or an action is destructive. (3) **Finish the job** — pursue the outcome until verified or genuinely blocked, don't stop at diagnosis or a plan. (4) **Protect existing work** — preserve user changes and other agents' work, never amend commits or reset state without authorization. (5) **Verify reality** — test behavior and contracts, not source text or config tautologies; never claim success without fresh evidence. (6) **Communicate for humans** — lead with the outcome, explain decisions and risks rather than mechanics. (7) **Learn in the right place** — durable guidance goes in `AGENTS.md` (with `CLAUDE.md` importing it), not in agent-private memories or quoted-text triggers.

The document reads as the distilled residue of the same failure modes the broader wiki tracks — over-eager mutation, diagnosis-without-completion, unverified success claims, and instructions that live in the wrong layer.
