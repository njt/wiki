---
url: https://github.com/lhl/devstack/blob/main/writing/20260703-ai-value-chain.md
title: Frontier Labs, Enterprises, and the AI Value Chain
author: lhl
date_fetched: 2026-07-05
date_published: 2026-07-03
---

The document examines whether AI labs extract durable value from enterprise customers during deployment engagements, and where defensible value sits across the AI stack.

## Structure

### Section 1 — Scope and Claims
Opens with the "suspicion" that labs like OpenAI (via "DeployCo," a subsidiary with $4B and embedded engineers), Anthropic (a reported $1.5B JV with Blackstone/Goldman Sachs/Hellman & Friedman), Microsoft, and Amazon send forward deployed engineers (FDEs) inside customer companies. These engineers learn internal workflows, judgment calls, and data — knowledge that may improve the lab's models in ways customers never see.

Two anchoring events: **Windsurf** (a coding-assistant startup whose Claude access was cut by Anthropic during acquisition talks with OpenAI) and **Bridgewater** (a hedge fund that tested frontier models at ~50% on its own document tasks, then fine-tuned an open Chinese model to 84.7% at ~1/14th the cost).

### Sections 2–3 — Three Separate Questions and What Each Side Owns

1. **The training question** — Are labs secretly training on enterprise data? Verdict: Weak today. Contracts across all four major providers say no by default. Azure adds architectural separation. But defaults can change.
2. **The byproducts question** — Do FDE engagements produce reusable artifacts (tasks, graders, rubrics, expert decisions)? Verdict: Strong.
3. **The competition question** — Can labs use platform visibility and access control to enter their customers' markets? Verdict: Strong.

### Sections 4–6 — Why Labs Are Doing This

Frontier-model economics don't close at the model layer. Training costs tens of billions; raw model access prices are collapsing (GLM 5.2 at ~$1.40/M tokens vs. Opus 4.8 at $5/M; Coinbase projecting 80% of workloads on 99%-cheaper models in 12–18 months). So labs move up the stack: services (DeployCo), first-party apps (Claude Code, Cowork), and the "discovery layer" (Claude Science for drug discovery).

### Sections 7–9 — Platform Competition and Customer Countermeasures

Section 7 documents the competition question: Google/OpenAI/Anthropic controlled ~90% of the $37B enterprise model-access market. The Windsurf cutoff, Anthropic cutting off OpenAI's API, and Cowork-induced selloffs are cited. The Vanderbilt Policy Accelerator's "AI Neutrality" proposal (modeled on common-carrier law) is noted.

Section 8 covers four customer countermeasures: Bridgewater (fine-tuned open model outperforms frontier), Coinbase (single gateway, 91% of engineers never hit usage caps, spend down 50%), sovereignty (France's DGSI, Germany's BfV, Spain, UK), and the gateway pattern (single internal LLM routing layer).

Section 9 examines the training question in depth with a table mapping each data channel (ordinary usage, safety logs, fine-tuning data, opt-in programs) and whether it feeds training. Two recent developments: the **Mythos-class data channel** (Anthropic's Fable 5 requires 30-day retention and `provider_data_share` even through hyperscalers) and the **Claude Code steganography episode**.

### Sections 10–12 — Equilibrium, Enterprise Playbook, Evidence Grading

Section 10 describes a "stratified truce": usage and learning both stratify by difficulty and by ownership. Frontier models keep the hardest tier; open/custom models absorb high-volume work. Four things would break the truce.

Section 11 offers an enterprise playbook with contractual controls (derivative-use restrictions beyond "no training," artifact ownership, research/product firewalls, per-endpoint retention audits, identity exposure decisions, API nondiscrimination clauses) and architectural controls (own the gateway, classify workflows into three regimes, treat expert labels as owned assets).

Section 12 grades each claim's evidentiary strength. Among the strongest claims: enterprise no-training commitments (grade: Strong), FDE byproducts (Strong), platform control against app-layer rivals (Strong), Bridgewater results (Strong), and sovereignty pushback (Strong).

## Key Quotes

- "OpenAI's DeployCo is an explicit move away from the platform-and-call-it-neutral position"
- "Everything the model sees, the lab sees — and forward-deployed engineers are the lab's eyes on systems no API call would ever reach"
- "Bridgewater tested frontier models at ~50% on its own document tasks, then fine-tuned an open Chinese model to 84.7% at ~1/14th the cost — a result that suggests frontier scaling may have saturated on tasks requiring tacit organizational judgment"
- "Coinbase projects 80% of workloads running on models 99% cheaper within 12–18 months"
- "The gateway is an enterprise's demand-aggregation surface — whoever owns it owns the data flywheel that makes every subsequent model choice downstream"
- "If continual learning is solved, the firewall between deployment and training disappears: every log, every decision, every expert override becomes high-value signal"

## Evidence Grades

| Claim | Grade |
|-------|-------|
| Enterprise no-training commitments | Strong |
| FDE byproducts are reusable artifacts | Strong |
| Platform control against app-layer rivals | Strong |
| Bridgewater fine-tuning results | Strong |
| Sovereignty pushback | Strong |
| Enterprise content flows into shared frontier weights today | Weak |
| Continual learning makes deployment logs decisively more valuable | Unresolved |
