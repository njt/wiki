---
url: https://www.allthingsdistributed.com/2026/11/aws-emulator-and-the-engineers-job-when-agents-write-the-code.html
title: "AWS Emulator and the engineer's job when agents write the code"
author: Swami Sivasubramanian (guest post on Werner Vogels' blog, with an intro by Vogels)
date_fetched: 2026-11-13
date_published: 2026-11-13
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Launch-day essay for AWS Emulator, a tool that gives coding agents a local copy of AWS to test against, free and offline, built entirely by agents from 400+ Smithy API models covering 200+ services, released under CC0 with public commit history and a "things-that-do-not-work" folder of known gaps.

The essay frames the emulator with history: twenty years ago Sivasubramanian built Alf, a Paxos-based distributed lock service (ancestor of the replication behind S3 and DynamoDB), which Amazon could only make trustworthy by hand-building a JVM simulation of its environment — dropping packets, injecting faults, replaying concurrency orderings — plus formal reasoning with TLA+. Simulation worked when a small team could hand-build one system's environment; with 200+ AWS services, hand-built test environments became impossible, and agents now write code faster than any team can check it in production.

The idea (from Matt Goldberg) was to have agents build and test the environments themselves: agents write fidelity tests from public API models, verify them against real AWS, then build the emulator until it passes those same tests. Kiro produced a passable SQS emulator from one prompt in about 15 minutes; every change goes through a merge gate of automated checks including security agents doing adversarial reviews.

The core argument for the wiki: when agents write and check their own code, the engineer's job is to build the specs, tests, and merge gates that decide when the work is correct. Sivasubramanian illustrates this with his own mistake — supervising an agent rebuilding Alf's replication engine for four nights without giving it a way to check its results; once it had a spec and a test environment, the rebuild took about two hours versus the four months of the original.
