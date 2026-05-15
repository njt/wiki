# Chief of Staff

Doneyli De Jesus built an AI chief of staff that manages email, calendars, and family logistics across four accounts. The architecture lesson: not everything needs to go through an LLM. A two-tier system (rule-based scanning every 30 minutes, LLM classification once daily) cut API costs by 80%. Trust is graduated, measurable, and revocable -- a rolling 90-day window, not a permanent achievement.

---

## Key Quotes

> "When your agent sends an email it shouldn't have at 2 AM, 'I don't know how this layer works' is not acceptable."

## Key Themes

#agent-architecture #trust #automation #cost-optimization #personal-ai

This is the most practically useful piece in this batch for anyone building agents that act on behalf of humans. Three patterns worth stealing:

**Deterministic fast paths:** Route 80% of inputs through rule-based systems (zero LLM cost); reserve the LLM for the 20% requiring judgment. This is the opposite of the "throw everything at the LLM" approach and it works.

**Graduated autonomy:** Three levels with measurable graduation criteria, plus hardcoded safety exceptions for VIP/family contacts regardless of confidence scores. Trust can be revoked. This directly implements what [[Experience Design for Agents]] calls "progressive authority."

**Three-layer memory:** Observations (zero-cost logging) feed into Memories (daily LLM synthesis) accessed via Retrieval (BM25, capped at 550 tokens). Memory decay prevents context pollution. This is a concrete implementation of what [[Elements of Agentic Systems Design]] calls the Memory and Learning elements.

The technical stack is refreshingly scrappy: a repurposed MacBook Pro, Docker, Python, launchd, Tailscale, Signal. 313 commits, 43K lines of Python, $100/month in LLM costs. This is what real agent deployment looks like -- not a demo, not a startup pitch, just a person solving their own problem.

## Critical Analysis

Strong: the cost optimization through tiered processing is immediately applicable. The graduated autonomy model with measurable criteria is the best implementation I've seen of the "earned trust" pattern. The memory architecture with explicit decay functions addresses the context pollution problem that kills long-running agents.

Missing: the article is long on implementation detail but short on failure cases. What emails did the system send that it shouldn't have? What did the graduated autonomy model catch? The 80% send rate accuracy stat suggests 20% of drafts get edited, but we don't know the nature of the failures.

The "why I didn't use OpenClaw" section is telling -- he wanted to understand every layer because agent mistakes at 2 AM are his problem. This is the [[yolo-cage]] philosophy applied to personal agents: the blast radius matters.

---
*Sources: [[raw/chief-of-staff]]*
*Last updated: 2026-05-14*
