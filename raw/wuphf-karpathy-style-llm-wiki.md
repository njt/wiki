---
url: https://news.ycombinator.com/item?id=47899844
title: "Show HN: A Karpathy-style LLM wiki your agents maintain (Markdown and Git)"
author: najmuzzaman
date_fetched: 2026-05-15
date_published: 2026-05-15
tags: [agent-wiki, llm, markdown, git, bm25, sqlite, knowledge-management, wuphf]
---

# Show HN: A Karpathy-style LLM wiki your agents maintain (Markdown and Git)

**Posted by:** najmuzzaman | **260 points** | **115 comments**

## Original Post

The author shipped a wiki layer for AI agents using markdown + git as the source of truth, with a bleve (BM25) + SQLite index. It runs locally at `~/.wuphf/wiki/` and is git-cloneable.

Key features:
- Each agent gets a private notebook at `agents/{slug}/notebook/.md` plus shared team wiki access
- Draft-to-wiki promotion flow with back-links, expiry, and auto-archive
- Per-entity append-only fact log (`team/entities/{kind}-{slug}.facts.jsonl`) with a synthesis worker that rebuilds entity briefs. Commits use "Pam the Archivist" git identity
- [[Wikilinks]] with broken-link detection (red rendering)
- Daily lint cron for contradictions, stale entries, broken links
- `/lookup` slash command + MCP tool; heuristic classifier routes short lookups to BM25 and narrative queries to cited-answer

Substrate: Markdown for durability. Bleve for BM25, SQLite for structured metadata. No vectors yet. Benchmark: 85% recall@20 on BM25 alone.

Known limits: "Recall tuning is ongoing. 85% on the benchmark is not a universal guarantee." "Synthesis quality is bounded by agent observation quality." Single-office scope only.

Context: Ships as part of WUPHF, an open source collaborative office for AI agents. "MIT, self-hosted, bring-your-own keys."

Links: GitHub: https://github.com/nex-crm/wuphf | Install: `npx wuphf@latest`

## Selected Comments

**portly**: "The whole point of taking notes for me is to read a source critically, fit it in my mental model." Feels copy-paste at 100x scale misses the point.

**frocodillo** (reply): Notes this is a harness for coordinating agent work, not just note-taking. Shared using Obsidian with AI help for structuring notes: "I can just jot down random thoughts and ideas, and the agent helps me structure it."

**bushido** (reply): Uses an agent harness with a full team. "I spend most of my time fine-tuning the harness." Reports team producing at 5x capacity from three months ago.

**simsla**: "Everyone is writing. Nobody is reading."

**mohamedkoubaa**: From a decade of engineering experience: "writing code was always a smaller fraction of my time compared to reading code, debating code with colleagues, and wrangling ops."

**skybrian**: "The LLM's do quite a lot of reading. The question is what to feed them."

**nicbou**: "It's such a promising technology, but it seems like the primary use case is to drown everything in noise."

**4b11b4**: Called it "akin to eating without thinking... just a waste, no nutrition."

**stingraycharles**: Cited studies showing "a *degradation* of output quality when these markdown collections are fully LLM maintained." Believes human curation is essential.

**saadn92**: Runs a variation for 6 months. Extracts decisions/rejected approaches from transcripts. "I review before I promote it into the context."

**mplappert**: "I think there's a serious issue with people using AI to do an immense amount of busywork and then never look at it again."

**zby**: Posted a review of WUPHF. Noted it's the third LLM wiki on front page in 24 hours. Shared a design wishlist. "I wish there was a chance for collaboration."

**Myrmornis** (reply to zby): Noted zby's notes "were written by an LLM" based on prose style. Asked if zby plans to rewrite in own words.

**zby** (reply): Iterates and cleans design notes. Reviews are auto-generated. The KB has a disclaimer: it's "agent-operated: a human directs the inquiry, AI agents draft, connect, and maintain the notes."

**SOLAR_FIELDS**: "this stuff is now in roll your own territory." Suggested QMD on an Obsidian vault gets you 80% there.

**jermolene**: Creator of TiddlyWiki. Shared twillm using TiddlyWiki's Node.js config. "Computed views replace materialised index files" — avoids LLM staleness problems with index.md.

**Abby_101**: Wanted the "garbage facts in, garbage briefs out" caveat stress-tested. "Six months in, you have entries that are confidently wrong." Asked if promotion flow requires human review.

**renan_warmling**: Pointed out missing snapshots and data enrichment. Raised integrity/consistency concerns and the need to distinguish temporal vs. atemporal memory.

**GistNoesis**: Shared Shoggoth.db project. Building self-organizing SQLite database.

**dataviz1000**: Argued LLMs are probabilistic: "the longer an agent runs on a task, the more likely it will fail." Suggests limiting run time: "It is better to let them run twice for 5 minutes than once for 10 minutes."

**iterateoften** (reply): "agents and ML reach local maximima unless external feedback is given. So your wiki will reach a state and get stuck there."

**ryanshrott**: Suggested "separating the capture layer from the promotion layer." Some teams use voting where "multiple agents independently summarize the same source."

**drewbatcheller** (reply): "The 'draft freely, promote on approval' method is the only thing I think works." Recommended a reviewer agent with memory.

**psanchez**: Built a git-based knowledge base for their company. "When the AI failed... each failure improved the system, and its capabilities compounded very quickly."

**imafish**: "Cool idea. But is anyone actually building real stuff like this with any kind of high quality?" Called "team of agents" claims "AI slop."

**stavros** (reply): "The problem with your comment is that the word 'real' is just there to move the goalposts."

**najmuzzaman** (reply to imafish): Their team: "4 HubSpotters who built HubSpot's largest platforms." Building enterprise context infra. Former HubSpot PM, backed by Dharmesh Shah.

**johntash**: Asked about keeping LLMs from writing too much, causing mess. An experiment "looked good until you read the actual data."

**mncharity**: Noted TiddlyWiki community exploring AI tooling. "a single-file agentic wiki wandering around and self-editing is feasible today."

**hansmayer**: Asked about making the starting directory configurable. When najmuzzaman filed the issue: "I mean did you really need someone on HN to tell you that?"

**najmuzzaman** (reply): "nicht geil. are you really so full of yourself?" Explained not every customization is required from day-1.

**Unsponsoredio**: "love the bm25-first call over vector dbs. most teams jump to vectors before measuring anything."

**davedigerati**: "why not an Obsidian vault with a plugin?"

**najmuzzaman** (reply): Two reasons: 1) Obsidian is single-user, lacks agent promotion state machine. 2) Agents need MCP surface, not plugin API. "you can absolutely point Obsidian at ~/.wuphf/wiki/ and use it as a vault."

**smadam9**: Building Mac app "Sig" around this concept: "capture has to come from you... That articulation is the work."

**tomjwxf**: Suggested the better signal for query routing is "the surrounding intent context the agent was operating under, not the query string itself."

**michelhabib**: "Just wondering if AI Notes would add value or create noise."
