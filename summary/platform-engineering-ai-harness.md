---
url: https://thenewstack.io/platform-engineering-ai-harness/
title: "Every Software Company Will Become a Dev Tools Company"
author: The New Stack
date_fetched: 2026-08-07
topics:
  - agent-coding-workflow
---

The article argues that platform engineering is absorbing AI toolchain ownership and becoming the control plane of software organizations. As AI generates more application code, the team responsible for building the machinery to control, verify, and scale that code generation is platform engineering. "Harness engineering" — building the feedback loops, guardrails, and context agents need to work safely — is not a new discipline but platform engineering with an AI-specific layer.

The scope of what platform engineering now owns is expanding: model and framework approval (security and cost implications), usage scaling (rolling out to hundreds of engineers differs from ten), cost governance (unmanaged usage gets expensive fast), authorization (agents acting on behalf of engineers need different access levels), and the feedback loops that route linter and security scanner findings back into the agent rather than just failing builds.

The article draws on a conversation with ThoughtWorks consultant Vanitha Kumar, who observed that organizations are drawing two separate "platform" boxes — traditional CI/CD infrastructure and a newer "agentic developer platform" — and realizing they're converging into one. The platform's reach is moving earlier in the lifecycle: it used to start at the first commit; now it starts before any code exists.

A good harness raises the odds the agent gets the task right on the first pass and gives it a way to catch and fix its own mistakes before a human ever sees them. Skip that work, and agents keep making the same mistakes forever. The article's central prediction: in many orgs, more engineers will end up on the platform side building and maintaining the harness than on the product side, because the harness is now the harder part of the job.
