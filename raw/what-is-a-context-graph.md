---
url: https://nanonets.com/blog/what-is-a-context-graph/
title: "Context graphs: how AI agents remember why decisions were made"
author: Karan (Kalra)
date_fetched: 2026-07-05
date_published: 2026-07-01
site: Nanonets Blog
---

# Context graphs: how AI agents remember why decisions were made

**Author:** Karan (Kalra)
**Published:** July 01, 2026 on Nanonets Blog

## Core Problem

The article opens with a detailed scenario about a renewal agent handling a $480k account where a customer wants 20% off or they'll leave, but policy caps renewals at 10%. A human would recall past exceptions (e.g., a similar Globex situation where leadership approved above-cap discounts), but the AI agent can't access the *reasoning* behind past decisions. Salesforce shows *what* happened (20% discount) but not *why*.

The author states organizations lose billions through "making the same mistakes, reinventing the same solutions, wasting time on previously solved problems," slow onboarding, and compliance gaps. The root cause: "we have gotten extremely good at recording **what** happened, but we systematically throw away **why** it happened."

## Flat Context Fails

### Context rot
When processing invoice #842, an agent must hop across the invoice, PO, budget, delivery receipt, vendor hold status, approval thresholds, contract terms, policy docs, and Slack/Zoom nuance. Dumping all this as disconnected text forces the LLM to re-derive connections from scratch each time. As context grows large, the article cites the well-known "lost-in-the-middle" phenomenon and notes that Surge AI's benchmark shows "The best frontier model solves <41% of such complex tasks."

### Lack of decision traces
Tribal knowledge, past deal structures, cross-system context, and manual approvals all contain crucial *why* information that never gets captured. The article analogizes to Architecture Decision Records (ADRs), "invented back in 2011 to fix exactly this. But most ADR folders die at three entries."

## What Is a Context Graph?

"A context graph is a way of structuring an agent's memory as a graph, where nodes hold pieces of information and edges hold the relationships between them." It's "optimized for the agent to read, not for a human to browse."

Unlike standard vector RAG (which returns similar text chunks with no connection info), a context graph keeps typed edges: "Service A –depends on–> on Service B," or "this invoice –follows–> that policy." This matters because "similarity is not relevance."

Foundation Capital's phrasing is cited: a "system of record for decisions." Systems store current state; the context graph stores "how the state got that way."

## How to Create One

Entities (invoice, PO, account, vendor, contract, policy, approver) become nodes. Relationships become edges. For each task, the agent pulls a small subgraph, keeping context windows small.

Decision traces become the unit of storage. A flat record says "Initech renewed at 20%." A decision trace includes "the problem that triggered it, the options weighed, why the rejected ones lost, the constraints, the exceptions, who decided, and the reasoning."

## How to Use One

### Capture on the way in
Reconstructing context later is "lossy guesswork." Capture the decision *when it's made* — the context is already in the active window. If a human overrides the agent, that's "the moment to ask why and store the answer with minimal friction."

The article argues agents change the economics of organizational memory. Wikis, Confluence, post-mortems, and ADRs all decay because "writing it down is friction, and nobody reads it back." Agents break both problems: "capture is a side effect of doing the work" and the agent is "a tireless reader that will happily consult ten thousand past decisions."

### Use stored decisions as precedent
The agent pulls direct context, finds closest precedents via vector embeddings, reasons across them, takes a decision, stores a trace, and links it to similar past decisions. This creates a "mode of self-learning without anyone fine-tuning it."

### Pattern detection
If the Net-60 exception keeps getting granted to twenty vendors, "the policy is wrong, not the vendors." The graph surfaces signals to fix underlying policies.

The article cites the ACE paper ("Agentic Context Engineering," arXiv 2510.04618): treat accumulated context as "a playbook that grows through generation, reflection, and curation" — "A correction today becomes a rule tomorrow. A trace today becomes precedent next quarter."

## Context Graph vs. Knowledge Graphs

"Knowledge graphs have been around since Google shipped one in 2012." Event sourcing is a familiar backend pattern. A context graph is "close to event sourcing for decisions" where each event carries its rationale and links. "What's new is that you capture the why on the write path as structured data" because there's now an agent hungry to read it.

## Isn't Agentic Search Enough?

The article acknowledges agentic search works well — citing the AgenticRAG paper showing 49.6% recall vs 8.4% single-shot on BRIGHT, and 92% answer correctness on FinanceBench. But two problems remain:

1. **Cost**: "That 92% on FinanceBench cost 115K tokens per query, about 8x the single-shot cost." The agent re-derives the same links daily. A context graph uses memoization — store once, traverse instead of re-deriving. Papers cited: A2RAG cut tokens/latency ~50% with +10 recall points; GRASP got highest multi-hop accuracy with 40-50% fewer tokens.

2. **Agentic search can only find what was written down** — "No search method fixes a write-path problem." The Initech decision reasoning "lived in a Zoom call and three Slack replies, and half of it never left anyone's head."

## System-of-Record Agents Won't Work

Salesforce's Agentforce, ServiceNow's Now Assist, Workday's agents "will inherit the exact same limitations as their parents": they capture *what* changed, not *why*; and each system misses data outside its domain. "No single system of record sees the whole picture." The article argues the orchestration layer alone sees full context — "what inputs were gathered, what policies applied, what exceptions were granted" — and therefore can capture that context at decision time.

## The Hard Parts

1. **Garbage in, garbage precedent** — "The graph is worth exactly the quality of the why you put in it."
2. **Who writes the trace** — Human typing rationale risks wiki-like decay; model-inferred rationale is shaky. "Getting that right is not trivial."
3. **The decision swamp** — "A graph of millions of contradictory, half-true traces is the same failure with extra edges."
4. **This is early** — "Most vendor decks make it sound shipped. It isn't." Great early results aren't proven.

## The Full Stack

Four layers:

1. **Systems of record** — Salesforce, SAP, Zendesk, GitHub, Slack. "What they don't hold is the reasoning that connects them."
2. **The harness** — Runs the reason→act→observe loop, holds tools, stores corrections as memory, enforces permissions, logs decisions.
3. **The context graph** — Every decision leaves a trace: inputs, rule applied, exception taken, who approved, why.
4. **Agents and humans** — Agents handle routine cases; humans handle uncertain ones; corrections flow back into memory.

This maps to two features: **Universal context** (systems of record made queryable through the graph) and **Loops** (harness closing feedback on every run).

## Where to Start

Build a context graph when agents run long, decisions must survive many turns, and questions chain facts. The article suggests processes with: high team size (50+ people running a manual workflow), exception-heavy decisions (procurement, insurance claims, deal desks, compliance), and cross-functional roles (RevOps, FinOps, DevOps) — roles that "emerge precisely because no single system of record owns the cross-functional workflow."

## Closing Analogy

"The model is your brain, the agent / agentic harness is your limbs, and the context graph is the map of your specific world (or company)."
