---
url: https://www.preemptive.com/blog/ai-security-framework/
title: "How to Build an AI Security Framework for DevSecOps Teams"
author: Michelle Pruitt
date_fetched: 2026-07-18
date_published: 2026-07-16
topics:
  - guardrails-and-feedback-loops
  - security-and-sandboxing
---

A guide to building a practical AI security framework for DevSecOps teams, published by PreEmptive. It argues that traditional AppSec — SAST, SCA, endpoint scanning — doesn't cover AI-specific risks like prompt injection, insecure output handling, model supply chain issues, and adversarial abuse. The response is a lifecycle-wide framework spanning governance, model risk, data and prompt security, application hardening, and runtime controls.

The article maps five major reference frameworks — NIST AI RMF, the EU AI Act, OWASP Top 10 for LLMs, MITRE ATLAS, and Google SAIF — and positions them as complementary rather than interchangeable. NIST and EU AI Act sit at the governance/regulatory level; OWASP and MITRE ATLAS provide technical threat guidance; Google SAIF offers implementation principles.

Four core principles anchor the framework: secure-by-design (security baked into architecture, not bolted on), layered defense (no single control is enough for non-deterministic AI systems), lifecycle coverage (design through operations), and workflow-friendly enforcement (controls embedded in CI/CD, not optional manual steps).

The top risks it flags are prompt injection, sensitive data exposure through AI pipelines, reverse engineering of AI-enabled binaries, runtime tampering, and uncontrolled agent actions. Controls are organized across five lifecycle stages: design/architecture, development/integration, build/release, deployment/runtime, and ongoing review.

A self-interested section positions PreEmptive's products — obfuscation, hardening, anti-tamper — as the application-layer piece of the puzzle for .NET, MAUI, Java, Android/Kotlin, and JavaScript applications. The strongest insight is that AI security frameworks fail when they stay at the governance level without concrete, enforceable controls wired into the delivery pipeline.
