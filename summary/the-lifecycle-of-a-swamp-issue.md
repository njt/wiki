---
url: https://stack72.dev/the-lifecycle-of-a-swamp-issue/
title: "The Lifecycle of a Swamp Issue"
author: Paul Stack
date_fetched: 2026-05-15
date_published: 2026-04-08
publication: "The View from the AI Frontier (stack72.dev)"
tags: Swamp, agent-workflow, issue-lifecycle, state-machine
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

# The Lifecycle of a Swamp Issue

Paul Stack describes how the Swamp team uses their own tool, Swamp, to manage the development process for Swamp itself. An issue is not a traditional GitHub thread but an instance of a model called `@swamp/issue-lifecycle` — a state machine with versioned plans, adversarial reviews, and enforced checks.

## The Five Phases

1. **Triage** — The issue is fetched from swamp-club, the agent reads the codebase, reproduces bugs if applicable, and classifies it (bug, feature, or security).
2. **Planning** — The agent drafts an implementation plan against the repo's planning conventions, breaking down steps, files, testing strategy, and risks.
3. **Adversarial Review** — The plan is critiqued across repo-specific dimensions; findings are recorded with severities.
4. **Iteration** — Human feedback drives revisions; review runs again. Loops until the plan is clean and approved.
5. **Implementation** — Only after approval does the agent begin the actual work.

## Why GitHub Was Cut Out

As the lifecycle grew to include structured state (plan versions, findings, feedback rounds, phase transitions), GitHub issues became inadequate. The team tried keeping GitHub as the system of record but found that "the real state lived in the swamp datastore and GitHub was just a reflection of it." This created sync problems. Swamp-club became the source of truth, itself a swamp model, where each method call posts a structured lifecycle entry as part of the same step.

## Key Design Decisions

- **Resumability**: Any agent can join mid-process, inspect the current state via `swamp model get issue-N --json`, and see the next valid transition. Model-level checks prevent mistakes.
- **Auditability**: All plan versions, feedback rounds, findings, and state transitions are timestamped data records.
- **Adaptability**: Each repo can place files in `agent-constraints/` to tailor the lifecycle without modifying the core model.
- **Operator-driven triage**: Nothing happens automatically when an issue is filed. An operator kicks things off — partly to prevent prompt injection attacks via issue bodies, and partly so a human who will live with the outcome frames the context.
- **No auto-approval of plans**: The agent generates, reviews, and presents, but "the human reads it and either gives feedback... or says to proceed." That rule applies during planning; implementation has a separate verification process covered in a promised follow-up post.

## Dogfooding

The issue lifecycle model itself runs on swamp infrastructure. When the team wants to change triage behavior, they edit the `issue-lifecycle` extension, and "the upgrade system carries old instances forward." Conventions are changed by editing files in `agent-constraints/`. The author frames this bluntly: "If the tool can't handle its own process, it isn't ready to handle yours."

## State Machine Guards

Methods like `approve`, `implement`, and `triage` are guarded by checks. Approving a plan requires one to exist, have undergone adversarial review, and have no unresolved critical or high findings. Calling methods in the wrong phase simply bounces. "The agent physically cannot skip steps."

## Related Posts

- "The Rearchitecture Your Team Could Never Justify" (14 May 2026)
- "When Building Is Easy, the Danger Is Building Everything" (11 May 2026)
- "Take the Leap Before You Are Pushed" (06 May 2026)
