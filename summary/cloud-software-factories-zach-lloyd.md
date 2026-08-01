---
url: https://xcancel.com/i/article/2077056692987027758
title: "The guide to software factories"
author: Zach Lloyd
date_fetched: 2026-07-25
date_published: 2026-07-14
---

Zach Lloyd (CEO of Warp) lays out the case for cloud software factories — centralized, automated systems that run the SDLC loop (triage → spec → implement → review → verify → ship → monitor) with agents doing most of the work and humans steering as needed. The article is aimed at engineering leaders evaluating the shift from interactive coding agents to factory-style automation.

Lloyd argues that interactive agents (Copilot, Cursor, Claude Code) have hit a wall: every engineer uses them, but it's unclear whether token-plus-human cost has real ROI. They also create sprawl that's hard to govern — developers use different models, different prompting skill levels, and MCP setups that can introduce security risk. A cloud factory centralizes agent execution, standardizes environments, and makes cost and output measurable.

The factory has three layers. Layer 1 is the cloud runtime and sandbox — moving agents off laptops onto cloud hosts (AWS, GCS, Modal, Daytona) with a coding agent harness and MCP integrations. Layer 2 is orchestration: triggering agents from Slack tickets or Jira issues, managing multi-harness/multi-model routing, and providing human-in-the-loop primitives (steering a live session, handoff between cloud and local, notifications when an agent needs help). Layer 3 is measurement, evals, and memory — tracking factory efficiency as shipped product over token cost, running experiments to tune the pipeline, and building agent memory that the company owns.

He recommends defining factories as code (infra-as-code patterns) and advises most companies to buy rather than build, warning against vendor lock-in through single-model anchoring, data capture, compute inflexibility, or forced token reselling. Lloyd positions Warp as a vendor in this space but also points to alternatives. The piece closes by noting that about 20–30% of PRs are fully automatable today, with that number expected to rise rapidly.
