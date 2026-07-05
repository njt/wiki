# Wuphf — Karpathy-Style Agent Wiki

A markdown+git wiki substrate for teams of AI agents, built by four ex-HubSpot engineers as part of the WUPHF open-source collaborative office. Agents get private notebooks, shared team access, and a draft-to-wiki promotion flow with broken-link detection, daily lint crons, and BM25+SQLite retrieval. The third LLM wiki to hit the HN front page in 24 hours, and the discussion it triggered is more interesting than the tool itself.

---

## Key Quotes

> "Markdown for durability. The wiki outlives the runtime."

The core substrate bet. No vector DB, no proprietary format — just files in a directory under git. If WUPHF dies tomorrow, the wiki is still a directory of markdown you can open in Obsidian, grep, or clone. This is the same bet behind this wiki ([[LLM Wiki]]) and [[robot.wtf]].

> "Everyone is writing. Nobody is reading." — simsla

The comment that crystallized the HN thread's central anxiety. 115 comments, and the dominant sentiment wasn't about WUPHF's architecture — it was about whether agent-generated wikis are knowledge or noise. This maps directly onto the [[Write Only Code]] problem: when generation is near-zero-cost, the ratio of production to consumption inverts.

> "The LLM's do quite a lot of reading. The question is what to feed them." — skybrian

The counterpoint. The audience for agent wikis isn't humans — it's other agents. The wiki as context infrastructure, not reference material. This reframes the "nobody is reading" critique: the reader is the next agent invocation.

> "I review before I promote it into the context." — saadn92

The promotion gate as the human-agent interface. This is the pattern that makes agent wikis work: draft freely, promote on approval. It's the same mechanism behind [[jibrain Knowledge Architecture]]'s three-tier curation pipeline and [[Cosmo's Blog]]'s human-approves-AI-writes workflow.

> "the longer an agent runs on a task, the more likely it will fail. It is better to let them run twice for 5 minutes than once for 10 minutes." — dataviz1000

A heuristic with implications for wiki maintenance. Short, bounded agent runs with fresh context may produce better wiki entries than long-running synthesis jobs.

> "agents and ML reach local maxima unless external feedback is given. So your wiki will reach a state and get stuck there." — iterateoften

The stagnation risk. Without external input — new sources, human correction, cross-agent debate — an agent-maintained wiki converges to a fixed point and stops improving.

> "love the bm25-first call over vector dbs. most teams jump to vectors before measuring anything." — Unsponsoredio

A recurring theme in this wiki's pages ([[QMD]], [[Harness Engineering]]): measure before you architect. BM25 at 85% recall@20 is a solid baseline that most teams skip straight past.

---

## Key Themes

#tool #agent-wiki #context-infrastructure #knowledge-management #bm25 #markdown

### The Architecture

WUPHF's wiki layer has several design decisions worth noting:

- **BM25 + SQLite, no vectors.** BM25 for retrieval, SQLite for structured metadata. This is deliberately simple — 85% recall@20 as a baseline, with vector search as a future addition rather than a premature optimization. Contrast with [[QMD]] which combines BM25 + vector + LLM re-ranking, and [[CodeMira]] which uses SQLite + hnswlib + FTS5.

- **Draft-to-promote state machine.** Agents write drafts; humans (or reviewer agents) promote them to the wiki. This separates generation from curation and prevents the "garbage facts in, garbage briefs out" problem Abby_101 warned about. The same pattern appears in [[jibrain Knowledge Architecture]]'s reweave pass and [[Cosmo's Blog]]'s human approval gate.

- **Per-entity fact logs with synthesis worker.** Append-only JSONL fact files per entity, with a synthesis worker ("Pam the Archivist") that rebuilds entity briefs. This is a temporal memory model — facts accumulate, briefs are periodically regenerated. renan_warmling's critique about distinguishing temporal vs. atemporal memory is relevant here.

- **Daily lint cron.** Contradiction detection, stale entry flagging, broken link detection. This is the [[Feedback Loop is All You Need]] pattern applied to wiki maintenance — deterministic checks, not prompt-based hoping.

- **Heuristic query classifier.** Short queries route to BM25; narrative queries route to cited-answer generation. tomjwxf noted that intent context from the agent's task is a better routing signal than the query string itself.

### The HN Debate: Knowledge vs. Noise

The thread split into two camps:

**The skeptics** (portly, simsla, mplappert, nicbou, 4b11b4, stingraycharles, imafish): Agent-generated wikis are busywork at scale. The act of note-taking IS the thinking. Automating it produces well-formatted garbage that nobody reads. stingraycharles cited studies showing "degradation of output quality when these markdown collections are fully LLM maintained."

**The practitioners** (frocodillo, bushido, saadn92, psanchez, najmuzzaman): This isn't note-taking replacement — it's context infrastructure for agent teams. The wiki is the shared memory that lets agents coordinate across sessions. The human curates; agents do the filing.

Both sides are right about different things. The skeptics are right that agent-generated wikis degrade without human review. The practitioners are right that the alternative isn't a human-maintained wiki — it's no wiki at all. The question isn't "are agent wikis as good as human ones?" but "are agent wikis better than the context rot that happens without any wiki?"

### Context Infrastructure as a Category

WUPHF is part of an emerging category: tools that manage agent context as first-class infrastructure. This wiki itself is an instance of the same pattern. Other entries in this space:

- [[LLM Wiki]] — the Karpathy pattern this implements
- [[robot.wtf]] — git-backed wiki with MCP, human-agent symmetry
- [[jibrain Knowledge Architecture]] — three-tier production pipeline with seven-gate health audit
- [[Claude-Mem]] — automatic session capture and compression
- [[napkin]] — per-repo markdown scratchpad
- [[Planning With Files]] — filesystem as context persistence
- [[Three Tier Memory]] — hot/warm/cold tiered memory architecture

The common thread: context persistence is a systems problem, not a prompt engineering problem.

---

## Critical Analysis

WUPHF is substantively interesting but the HN thread is the real artifact here. The 115 comments form an accidental focus group on agent-maintained knowledge bases, and the anxieties they surface are more valuable than the tool's feature list.

**What works:**

The draft-to-promote flow is the right design. It acknowledges that agents produce slop and builds a gate rather than pretending the slop doesn't exist. The BM25-first approach is honest engineering — measure before optimizing. The daily lint cron is the linters-beat-prompts insight applied to wikis.

**What's concerning:**

The single-office scope limitation is significant. A team wiki that can't span teams is a knowledge silo. The "synthesis quality is bounded by agent observation quality" caveat buries the real problem: if your agents have poor observation, your wiki will be coherently, confidently wrong. Abby_101's "six months in, you have entries that are confidently wrong" is the nightmare scenario.

The hn discussion itself demonstrates the toxicity risk. hansmayer's "did you really need someone on HN to tell you that?" and najmuzzaman's "nicht geil" response show how quickly agent-wiki discussions become proxy wars about AI hype. The thread's most upvoted comments were the most skeptical — HN's immune response to anything marketed as "Karpathy-style."

**What's missing:**

No agent-to-agent debate or adversarial review. The synthesis worker is a single agent rebuilding briefs from fact logs — there's no mechanism for agents to challenge each other's entries. [[jibrain Knowledge Architecture]]'s reweave pass and ryanshrott's suggestion of "multiple agents independently summarize the same source" point toward a solution.

No snapshot or version comparison. renan_warmling's point about distinguishing temporal vs. atemporal memory is sharp. A wiki that can't tell you "here's what we believed in March vs. what we believe now" is missing half of knowledge management.

The configurable directory issue (hansmayer's complaint) is genuinely minor but the reaction to it was telling. najmuzzaman's defense — "every customization is a disaster if you did" from day one — is actually correct for early-stage products. Not every hardcoded path is technical debt.

**Bottom line:** WUPHF is a solid implementation of the Karpathy wiki pattern with genuinely good design decisions (BM25-first, draft-to-promote, daily lint). The HN thread it generated is a better artifact than the tool — a structured debate about whether agent-generated knowledge is knowledge at all. The answer, as with most things, is: it depends on the promotion gate.

---

*Sources: [[summary/wuphf-karpathy-style-llm-wiki]]*
*Source URL: https://news.ycombinator.com/item?id=47899844*
*Last updated: 2026-05-15*
