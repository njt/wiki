---
url: https://softwaredoug.com/blog/2026/08/29/ai-team-mistakes.html
title: "AI Team Mistakes"
author: Doug Turnbull
date_fetched: 2026-09-04
date_published: 2026-08-29
topics:
  - guardrails-and-feedback-loops
  - agent-memory-and-context
---

Doug Turnbull, a search/RAG consultant who has watched a dozen-plus budding AI
teams collide with reality, argues that today's AI teams are repeating the
mistakes search teams made in the 2010s. His core claim: **AI teams are search
teams**, and the pain points are structural, not superficial.

**Evals come first.** Great search/AI organizations spend roughly 50% of their
investment on *understanding* the problem rather than solving it. You can't
trust a PM's opinions about what "good" looks like; you eval instead. Evals find
where an agent fails, surface product-improvement opportunities, and become
training data. Turnbull cites Hamel Hussain and Shreya Shankar's evaluation
course, and his own Quepid (built 12 years ago) — because there is no objective
right/wrong answer in conversational systems. His Advanced Auto Parts anecdote
is the cautionary tale: employees searching a product wanted "what will I get an
incentive for selling?", not the product itself. The job isn't to build things;
it's to be a scientist — evaluate, hypothesize, test, improve.

**Retrieval is the whole thing, not a checkbox.** Teams assume one classic RAG
architecture fits everyone, but retrieval diversity is a blind spot. The
research finding is blunt: *retrieval dictates AI quality* — give the LLM the
right context and answer quality improves dramatically. Chunking, retrieval
tech, ranking, and result diversity all eat inordinate time, and it's easy to
sink cost into one approach. The skill is navigating cheap/dirty experiments
before committing to expensive/robust builds.

**Context means metadata, not chunks.** Classic RAG chunks raw passages and
searches by embedding similarity. Turnbull argues RAG is really about presenting
an agent *useful, trust-assessable* information. A chunk carrying title,
popularity, and publication date lets the LLM judge recency and trustworthiness
— a bare passage doesn't. Retrieval isn't just keyword/embeddings: representing
domain metadata, and giving agents tools to select on it, is a third, hidden
pillar (query understanding + metadata). Think "how do I represent a unit of
information and its provenance to an LLM," not "how do I chunk."

**Multidisciplinary teams.** Like search, AI teams thrive when they blend
scalable systems thinking with hypothesis-driven data science in one brain —
not data science throwing models over the wall to be discovered wrong three
months later.
