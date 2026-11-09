---
url: https://github.blog/ai-and-ml/github-copilot/beyond-code-generation-a-developers-guide-to-agentic-engineering/
title: "Beyond code generation: A developer's guide to agentic engineering"
author: GitHub Blog (staff, no byline)
date_fetched: 2026-11-09
date_published: 2026-10
topics:
  - agent-coding-workflow
  - ai-code-review
---

A vendor playbook from the GitHub blog arguing that coding got fast while everything around it didn't, and walking through a four-stage agentic workflow — Plan, Build, Verify, Observe — mapped to concrete Copilot features. Its opening statistics do the real arguing: GitHub saw 1.2 billion commits in a single October week (over 400% up year-on-year, more than all of 2025), yet a Faros AI study found pull requests spent 441% longer in review when teams moved from low to high AI adoption. The article's implicit thesis: the bottleneck has migrated from writing code to planning, reviewing, and operating it, and agents should be deployed across the whole lifecycle, not just generation.

The Plan section targets scattered context — design docs in one tool, requirements buried in Slack, rationale in old PRs — and proposes using Copilot in Slack/Teams, issue creation with acceptance criteria, agentic triage workflows, and Plan Mode for reviewing plans before implementation. Build leans on session continuity across devices, cloud sandboxes, isolated repo copies per task, repo instructions plus "Copilot Dreaming" (which reviews sessions and PR feedback to propose instruction/skill/memory updates), and model/effort selection to trade quality against cost.

Verify is the most substantive section: it names the 47-file "small refactor" PR as the review-queue pain in practice and prescribes stacked PRs, GitHub Code Quality checks, depth-matched Copilot code review, AI-based secret and vulnerability detection, license checks, and Agentic Autofix / Agent Merge to keep PRs moving — even auto-merging once checks pass. Observe closes the loop: connectors to observability and business data for incident investigation, drafting regression-test issues, and feeding lessons back into memories and skills. The advice lands on "start with what frustrates you most this week."
