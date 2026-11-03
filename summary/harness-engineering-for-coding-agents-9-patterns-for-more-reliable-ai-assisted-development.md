---
url: https://deepsense.ai/blog/harness-engineering-for-coding-agents-9-patterns-for-more-reliable-ai-assisted-development/
title: "Harness Engineering for Coding Agents — 9 Patterns for More Reliable AI-Assisted Development"
author: deepsense.ai team
date_fetched: 2026-11-03
date_published: 2026-10-31
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A practical playbook from deepsense.ai, drawing on their OpenAI Codex hackathon win with HyperCode (a repository-intelligence sidecar) and internal experiments with Codex and Claude Code. The core claim: "code is cheap, human time and model context are not" — reliability comes from engineering the system around the agent, not better prompts.

The nine patterns: (1) repo as knowledge base — a short AGENTS.md that documents *why*, with the deeper rationale stored in commit messages and regression-test docstrings; (2) just-in-time context assembled per task by a sidecar, over RAG against a living codebase; (3) durable tools and a few focused skills, promoted from recurring failures; (4) a discoverable validation menu in the task runner, shared with CI, that the agent is *told* to prefer (discovery alone proved insufficient); (5) bounded self-feedback loops — orchestrator/reviewer/fixer subagents with a retry cap, keeping the main context clean; (6) text-first UI validation, screenshots only when pixels are the requirement; (7) rewriting NEVER/ALWAYS prompt rules into intent + decision procedure + escape hatch, because rigid rules calibrated to an old model over-comply on a better one; (8) plan-mode pitfalls — approval promotes an incomplete plan into the task contract, so keep plans short and use cheap throwaway runs for low-risk work; (9) an "agentic garbage collector": capture friction during the session, surface it in the PR, and distill recurring mistakes into rules, tests, or linters — with human ratification.

Recurring meta-lessons: ambient self-reporting is unreliable, so capture must be enforced by harness checkpoints (hooks, PR gates); and the harness — not the prompt — should stay stable as models change.
