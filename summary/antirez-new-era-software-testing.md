---
url: https://antirez.com/news/168
title: "A new era for software testing"
author: "antirez (Salvatore Sanfilippo)"
date_fetched: 2026-06-12
date_published: 2026-06-08
topics:
  - software-engineering-craft
  - agent-coding-workflow
---

# A new era for software testing

antirez (Salvatore Sanfilippo, creator of Redis) argues that while AI-assisted coding can dramatically accelerate development, the output "does not reach the structural quality and economy of complexity of the best hand-written software." However, he notes that AI-generated code often surpasses "decently developed hand-written code," and there exists a tradeoff between quality and speed.

Where LLMs truly shine without compromise, he contends, is in software QA and testing.

## Traditional testing challenges

Test suites consist of local-scope tests and integration tests (using Redis as an example — verifying SET/GET behavior vs. testing replication correctness). Manual QA passes catch gaps in automated suites. He notes that "covering all the lines of the code does not mean covering all the possible states," and integration testing suffers from timing issues, setup complexity, and visual-only inspection requirements.

## The new LLM-based QA approach

Create a markdown file instructing an AI agent to act as a QA engineer on a new release. Using DwarfStar (an inference engine for open-weight LLMs) as an example, the agent is told to:

- Examine new commits since the last release
- Verify distributed inference across two MacBooks, checking output coherence and GGUF file compatibility on both machines
- Confirm no speed regressions exist

Notably, the speed regression check doesn't require pre-supplying previous benchmark numbers — that's a moving target across releases. The distributed inference test similarly needs minimal configuration (just SSH endpoints, keys, and paths in the file header).

The agent must review the QA checklist "especially in light of the added commits," starting by inspecting changes to identify affected areas, so the QA pass targets regression detection.

## Redis Arrays example

He used a similar methodology — instructing the agent to build a large array-based Redis application, set up a production environment with replication and persistence, then simulate multi-day, multi-user usage while monitoring for anomalies.

## Psychological/UX testing

The approach extends to qualitative aspects — asking the agent to identify features that seem surprising, under-documented, or "generally sloppy from the POV of the user." These were previously manual checks often skipped.

## Closing thought

antirez believes automatic QA may raise the quality bar for new software releases and "maybe partially compensate for the lower quality of the code produced at high speed with the use of automatic programming."
