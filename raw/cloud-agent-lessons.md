---
url: https://cursor.com/blog/cloud-agent-lessons
title: "What We've Learned Building Cloud Agents"
author: Josh Ma
date_fetched: 2026-07-11
date_published: 2026-06-02
---

# What we've learned building cloud agents

**Author:** Josh Ma
**Date:** June 2, 2026
**Publication:** Cursor Blog

Cloud agents (coding agents that run on remote infrastructure, not your laptop) are the next frontier. Cursor has been building them, and this post distills the engineering lessons: the development environment IS the product, long-running agents need durable execution, and you have to decouple agents from both machines and conversation state. The post is unusually concrete for an engineering blog — real numbers, named infrastructure choices (Temporal), and honest admissions about early reliability failures.

## The Development Environment is the Product

For local coding agents, the agent inherits your laptop: your shell config, your installed tools, your API keys, your repo state. For cloud agents, you have to reconstruct all of that from scratch. This isn't just about making things work — incomplete environments produce "a subtle degradation in output quality" rather than explicit errors. The agent doesn't crash; it just produces worse code.

> "When building a local coding agent, you inherit the user's existing development environment. It's set up the way they like it, with all their tools, configs, and credentials already in place. For cloud agents, you have to reconstruct the development environment on a remote machine, from scratch."

Cursor built what amounts to "enterprise IT for agents": VM hibernation/resumption, checkpoint/restore pipelines, secret redaction, network policies, credential management. The boring infrastructure work that determines whether cloud agents actually work.

## Long-Running Agents Need Durable Execution

> "Reliability is a threat vector in and of itself."

Cloud agents face failure modes that local agents don't: inference provider outages, pod replacements, EC2 node failures. Cursor's initial architecture was a work-stealing system that proved fragile — "our early beta of cloud agents often operated at one 9 of reliability."

The fix was migrating to Temporal for durable execution primitives (retry, scheduling, node failure durability). The migration "took us past two 9s of reliability." Today Temporal handles "more than 50 million actions per day across more than 7 million unique workflows."

> "Temporal gives us powerful primitives: durable timers, automatically retried activities, and the ability to continue execution even in the face of node failures."

The team also evolved their Temporal usage: they started with "eternal" workflows but moved to shorter, single-task workflows. The reason was practical — version upgrades are easier when workflows complete quickly rather than running for days.

Internally, over 40% of Cursor's PRs now come from cloud agents, and that number is growing.

## Decoupling Agents, Machines, and Conversation State

> "A cloud agent might start its work on one machine, migrate to another when a new task begins, and spawn sub-agents that run on entirely different infrastructure."

Cursor split the system into three independent components:
- **The agent loop** — runs in Temporal, owns the work
- **The machine state** — where execution happens, can migrate
- **The conversation state** — what the user sees, streams updates to clients

On the conversation side, they built an append-only storage layer that streams updates to clients and handles retries gracefully: "the client can detect this, rewind its stream, and show the new data."

This three-way decoupling is the engineering insight that makes the rest possible. Without it, you're building a monolith that can't survive infrastructure churn.

## Knowing How to Get Out of the Way

> "Early on, our agent harness would double-check its work after every task, force a commit, and push. Over time, we realized something counterintuitive: the more capable the model, the more the harness should get out of the way."

As the underlying models improved, Cursor steadily moved logic out of the harness and into tools the agent controls. This isn't about trusting the model more — it's about giving the model the agency to decide when verification is needed.

For CI Autofix, they now give the agent access to the GitHub CLI and write large outputs to searchable files rather than passing everything through context. The prompt philosophy also shifts: cloud agents get prompts encouraging more autonomy because "the cost of blocking is much higher" — a cloud agent waiting for human input could sit idle for hours.

## Self-Healing Agent Environments (Future Direction)

The current binary is unsatisfying: either you hand-hold the agent through environment setup, or you get out of its way and hope it doesn't hit a wall. Cursor's next goal is giving agents tools to understand and fix their own environments — "report when secrets are missing, network access is blocked" and act in a self-healing manner. The post references "autoinstall" as one path toward this.

This is the logical endpoint of "the development environment is the product": agents that can debug their own environment are agents that can operate without human babysitting.
