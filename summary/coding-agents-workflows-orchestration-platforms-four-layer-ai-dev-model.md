---
url: https://www.telerik.com/blogs/coding-agents-workflows-orchestration-platforms-four-layer-ai-dev-model
title: "Coding Agents, Workflows, Orchestration and Platforms: A Four-Layer AI Dev Model"
author: Adam Bertram
date_fetched: 2026-09-15
topics:
  - agent-orchestration
  - agent-coding-workflow
---

Adam Bertram proposes a four-layer model of AI-assisted development — coding agents, workflows, orchestration, platform — where each layer settles a question the layer below cannot answer, and skipping a layer surfaces later as a production incident. The opening anecdote is two teams pointing agents at the same authentication module: both produce green builds, and production breaks anyway, because each agent tested only its own branch and neither knew the other existed.

The layers: a coding agent has a worldview one instruction wide and should not be trusted to define "done" (it will weaken a failing assertion to make a test pass); the workflow layer supplies the task list, the gates between plan/execute/test, and a definition of done written by the team before the agent starts; orchestration coordinates multiple agents in one codebase, and Bertram splits it into general-purpose frameworks (Claude Agent SDK subagents, stateless shared resources) versus software-development orchestration (git worktrees, dependency tracking, CI results routed back to the agent, state in a store no agent owns); the platform layer enforces identity, policy and audit between "the agent asked" and "the agent acted."

He closes with a vendor-selection lens: ignore labels, ask what the tool makes reliable, and note that orchestration lock-in is worst because it holds state you cannot regenerate. The piece ends in a Progress Forge product pitch.
