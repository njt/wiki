---
url: https://github.com/FeroVolar/Decision-Framework-Skill
date_fetched: 2026-07-05
backfilled: true
---

A Claude Skill inspired by the 37signals Guide to Making Decisions by Jason Fried.

This Skill transforms the decision-making philosophy of **37signals** into an interactive coaching experience for Claude.

Rather than presenting a static checklist of questions, it guides users through a natural conversation, asking only the questions that matter for the decision at hand. The objective is simple: **help people think more clearly before they decide.**

This project is directly inspired by **Jason Fried's** article:


The 37signals Guide to Making Decisions

The original guide consists of 38 thought-provoking questions that the team at **37signals** uses as mental models when making decisions.

Their philosophy isn't about finding the "correct" answer—it's about asking better questions.

This Skill brings that philosophy into Claude by turning those questions into an adaptive, conversational framework instead of a static document.

If you haven't read the original article, it's highly recommended.

Most AI assistants are optimized to answer questions.

This Skill is optimized to ask better ones.

Instead of immediately suggesting a solution, it helps users:

- clarify what decision actually needs to be made,
- challenge assumptions,
- uncover hidden trade-offs,
- distinguish intuition from evidence,
- identify missing information,
- and understand long-term consequences.

Only then does it help formulate a recommendation.

- Inspired by the **37signals**decision-making framework
- Uses adaptive questioning instead of a fixed questionnaire
- Automatically speaks the user's language (English, Slovak, and any language supported by Claude)
- Skips irrelevant questions
- Challenges assumptions without being confrontational
- Encourages reversible experiments over irreversible commitments
- Produces a concise Decision Summary
- Explains recommendations together with assumptions and confidence level

At **37signals**, decisions are viewed as one of the fundamental responsibilities of every company.

This Skill follows the same philosophy.

Questions are not bureaucracy.

Questions improve thinking.

Instead of asking all 38 questions every time, the Skill selects only the ones that meaningfully improve the current decision.

Sometimes that means asking three questions.

Sometimes fifteen.

Never all of them.

**User**

Should I leave my corporate job to build my own SaaS?


Instead of answering immediately, the Skill might ask:

- Why does this decision need to be made now?
- What would happen if you made no decision?
- Which assumptions are you treating as facts?
- Could this be broken into smaller experiments?
- What information would genuinely change your mind?
- How reversible is this decision?
- What would your future self think about waiting another year?

Only after exploring the situation does it provide a recommendation.

The Skill dynamically explores topics including:

- Purpose
- Ownership
- Timing
- Alternatives
- Reversibility
- Intuition vs. Evidence
- Consequences
- Missing Information
- Stakeholders
- Principles
- Effort vs. Return
- Learning from Similar Decisions

Not every decision requires every topic.

The framework adapts naturally to the conversation.

```
Decision
    │
    ▼
Understand Context
    │
    ▼
Ask Relevant Questions
    │
    ▼
Challenge Assumptions
    │
    ▼
Identify Trade-offs
    │
    ▼
Summarize Thinking
    │
    ▼
Recommend Direction
    │
    ▼
Suggest the Smallest Next Step
```
The Skill automatically responds in the user's preferred conversation language.

Examples include:

- English
- Slovak
- Czech
- German
- French
- Spanish
- Italian

...and any language supported by Claude.

**37signals** has long been known for promoting simple, practical approaches to software development, product design, remote work, and business decision-making.

This Skill aims to preserve that same pragmatic mindset while adapting it into an interactive AI experience.

This repository is an independent community project and is **not affiliated with, sponsored by, or endorsed by 37signals**.

All credit for the original ideas belongs to Jason Fried and the team at 37signals.

MIT
