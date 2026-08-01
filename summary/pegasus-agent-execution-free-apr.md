---
url: https://dl.acm.org/doi/10.1145/3793655.3793739
title: "Execution-free Agentic Program Repair for Enterprise-Scale Development"
author: Saurabh Bodhe, Sanjukta De, Subhayan Roy, Jaydip Pokiya, Indira Vats, Sehajpreet Kaur, Xuemeng Li, Lejin Varghese, Yonas Bedasso, Max Kiehn
date_fetched: 2026-07-25
date_published: 2026-04-12
---

PegasusAgent is an execution-free agentic automated program repair (APR) system built at AMD on top of the Claude Agent SDK. Its core innovation is replacing test-based validation — impractical for industrial C++ codebases with long build cycles and proprietary test environments — with static analysis (CppCheck) combined with LLM-as-a-judge semantic critique.

The system runs end-to-end: it retrieves Jira tickets, performs a two-stage code localization (similar-ticket vector search followed by targeted GitHub code search), synthesizes patches informed by historical fix patterns, validates through CppCheck + LLM scoring, triggers a remote component build, and creates a pull request with auto-assigned reviewers. A planning-first ReAct loop drives the workflow via MCP-backed tools pruned to just six essential endpoints.

Evaluated on 112 real QML/C++ defects from AMD Radeon Software releases, the full system achieved 70.5% correct file localization and 33.1% plausible fixes (LLM-judge score ≥ 0.50). Ablation studies showed that response filtering was the single most impactful component: without it, localization collapsed to 23.2% and 46.4% of runs produced no patch at all. Tool filtering and similar-issue retrieval also contributed materially. Baseline Claude Agent SDK (without these optimizations) reached only 38.4% localization.

In a two-week production pilot with one team, 27.8% of defects received correct fixes and 11.1% resulted in merged PRs. The paper was published at FORGE '26, the IEEE/ACM conference on AI foundation models and software engineering.

---
*Sources: [[raw/pegasus-agent-execution-free-apr.md]]*
*Last updated: 2026-08-01*
