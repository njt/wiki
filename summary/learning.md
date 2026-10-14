---
url: https://asibackbone.github.io/Learning/
title: "AsiBackbone Learning — Practical .NET Architecture for Accountable Systems"
author: AsiBackbone project
date_fetched: 2026-10-14
date_published: unknown
topics:
  - agent-architecture
  - security-and-sandboxing
---

AsiBackbone Learning is a free, package-less .NET architecture education site teaching "governed execution": the idea that a proposed action should become an explicit, auditable decision before it becomes real-world execution. Its core model separates Intent, Context, Constraints, Decision, Decision receipt, Acknowledgment, Scoped authority, and Host-owned execution.

The material is organised into learning paths — Architecture, Governance, ASP.NET Core, Security, AI Integration, tutorials, labs, and executable samples — and follows a problem-first progression: problem, common implementation, failure mode, architectural pattern, teaching example, tradeoffs, working example. Notably, it also documents *when the patterns are unnecessary*: dedicated pages argue for "When ASP.NET Core Authorization Is Enough" and "When a Simple Application Service Is Enough", pushing back against applying governed-execution machinery everywhere.

The AI Integration path applies the same model to AI-assisted systems — tool calls, agents, workflows, host-controlled effects — so agent actions go through the same policy-evaluated, receipt-producing decision flow as ordinary privileged operations. Recent publications ("An Allowed Decision Does Not Mean the Operation Succeeded", "Idempotency Does Not Prevent Authority Replay", "Should Authorization Fail Open, Fail Closed, or Defer?") show a focus on the gaps between permission, effect, and replay.

The site explicitly disclaims being a compliance certification or security guarantee, frames working repositories as "specimens rather than unquestioned templates", and pins its baseline to AsiBackbone 7.0.0 even though 7.1.0 is newer.
