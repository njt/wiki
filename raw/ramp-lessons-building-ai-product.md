---
url: https://www.youtube.com/watch?v=NMs8C2_3M0w
gist: https://gist.github.com/93839a006849961f384afc60b1dcbf9f
title: "Ramp: Lessons from Building a New AI Product - The Pragmatic Summit"
speakers:
  - Nick (Ramp)
  - Veral (Ramp)
  - Will (Ramp)
  - Ian (Ramp)
channel: The Pragmatic Engineer
date_fetched: 2026-07-04
date_published: 2026-05-18
transcribed_by: njt
---

# Ramp: Lessons from Building a New AI Product - The Pragmatic Summit

Four Ramp engineers share hard-won lessons from building AI-native finance products for 50,000+ customers. The talk covers agent architecture, eval strategy, infrastructure, and the cultural shifts that matter more than the code.

## Key Points

1. **Single agent with many skills** — After experimenting with separate agents for every task, Ramp recognized the better model is "a single agent that can orchestrate thousands of tools and skills, triggered by events and guided by instructions, guardrails, context, and actions."

2. **Start small, let complexity emerge** — The Policy Agent began with a trivial case (coffee with a colleague) to learn what context truly matters — employee role, receipt details, HRIS fields — rather than trying to predict everything upfront.

3. **Tight feedback loops** — Ramp tested internally, then partnered with design customers (including a Fortune 500) in weekly meetings. They introduced an "autonomy slider" for moving from suggestions to auto-approvals, and let customers edit their expense policy like a `.cursorrules` file.

4. **Users are often wrong** — Finance approvers sometimes approve out-of-policy expenses out of laziness or trust. Ramp created a ground truth dataset through cross-functional labeling sessions.

5. **Evals are non-negotiable** — Starting with as few as five data points, making results interpretable, and integrating into CI. Offline evals catch regressions; online evals (e.g., "unsure" decision rates) provide live health checks.

6. **Infrastructure and culture both critical** — An internal "applied AI service" acts as an LLM proxy with structured output, batch processing, and cost tracing. A shared tool catalog (hundreds of tools, aiming for thousands) lets any team compose agent capabilities. Their internal coding agent Ramp Inspect now authors over 50% of merged PRs.

7. **Coding was never the hardest part** — The talk contrasts "Team A" (impact-obsessed, user-centric, embraces ambiguity) with "Team B" (bikesheds libraries, complains about headcount, builds before understanding). As AI handles implementation, judgment becomes the differentiator.

## Selected Quotes

- **Nick (on the paradigm shift):** "You don't need to build a thousand agents. … Instead you want to drive your framework towards a single agent with a thousand skills."
- **Veral (quoting Karpathy):** "English is the new programming language — and kind of turn the expense policy into the rules themselves."
- **Will (on user behavior):** "We thought that the users would be correct. … But turns out the users are actually incorrect. They're wrong."
- **Will (on the black-box tradeoff):** "A smaller black box becomes a bigger black box as the system becomes more complex."
- **Ian (on AI amplifying the real challenge):** "You could still build the wrong thing just a lot faster and you can build bigger messes."
- **Ian (on software being unfinished):** "Software is perpetually not finished. And so with all this extra capacity … companies are just going to chase opportunities they couldn't afford to pursue."

## Tools, Practices, and Methodologies

- **OmniChat** — Single conversational UX deployed across all product surfaces, replacing five separate chat interfaces.
- **In-house lightweight agent framework** — Orchestrates tools and deterministic workflows (playbooks) described in plain English.
- **Internal tool catalog** — Shared library of hundreds of APIs/actions any agent can invoke.
- **Applied AI service (LLM proxy)** — Centralized service with structured output, consistent SDKs, batch processing, cost tracing. Model swaps are one-line config changes.
- **Ramp Inspect** — Background coding agent using Modal sandboxes with full repo context, Datadog, read replicas, VS Code. "Multiplayer first" design; open-sourced at builders.ramp.com.
- **Streamlit-based labeling tool** — One-shot internal app for cross-functional labeling sessions.
- **Evals (offline + online)** — Offline: dataset starting with 5 examples, CI-integrated. Online: metrics like "unsure" decision rates.
- **Autonomy slider** — Product feature letting customers increase agent autonomy incrementally.
- **Expense policy as living document** — Customers edit policy directly and see immediate behavior changes.

## Unanswered Questions

- What happened on February 6th (a paradigm shift trigger)?
- How are hallucinations and financial errors actually prevented?
- Real costs of running all these agents
- Tool catalog management at scale
- Enterprise compliance and change management processes
- Maintainability of vibe-coded tools
- Specific failures or near-misses
- How the agent handles conflicting instructions or ambiguous policies
