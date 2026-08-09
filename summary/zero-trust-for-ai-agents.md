---
url: https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6a1611a04085d7cd3dadc924_Claude-eBook-Zero-Trust-for-AI-Agents-05182026.pdf
title: "Zero Trust for AI Agents"
author: Anthropic
date_fetched: 2026-08-09
date_published: 2026-05-18
---

Anthropic's guide to applying Zero Trust principles to autonomous AI agent
deployments in the enterprise. Written for CISOs and security architects,
organised into five parts: security considerations for autonomous systems,
current threats, the three-tier capability framework, an eight-phase
implementation workflow, and defensive operations at AI speed.

Three core Zero Trust principles anchor the framework: never trust and always
verify, assume breach, and least privilege. The guide extends these with *least
agency*, an OWASP-coined concept that restricts not just what an agent can
access but what each tool can do, how often, and where. A recurring design test
asks whether a control makes attack *impossible* or merely tedious — friction-
based defenses (rate limits, non-standard ports, SMS MFA) fail against agentic
attackers with unlimited patience at near-zero per-attempt cost.

The threat landscape covers prompt injection (direct and indirect), tool poisoning
and chaining, unscoped privilege inheritance in multi-agent systems, memory-based
credential retention, supply chain risks (Anthropic's research shows 250 malicious
documents can backdoor models up to 13B parameters, persisting through RLHF),
RAG poisoning, and shared-context poisoning.

The core framework applies three capability tiers — Foundation, Enterprise,
Advanced — across eleven domains including agent identity, access control,
privilege scoping, resource isolation, observability, behavioral monitoring, input
sanitization, output filtering, integrity and recovery, and AI governance. The
Foundation floor has been raised: short-lived tokens, cryptographically rooted
identity, identity-based isolation, and automated first-pass alert triage are now
minimum requirements. Static API keys and shared service-account passwords
"are no longer a legitimate entry point, not even at Foundation."

The eight-phase implementation workflow covers: requirements gathering,
supply chain risk management (AI-BOM, OpenSSF Scorecard, dependency
auditing, AI vendoring for unmaintained dependencies), agent boundary
definition with escalation triggers, prompt injection defense (Microsoft's
spotlighting cuts indirect injection success from >50% to <2%; Anthropic's
constitutional classifiers blocked 95% of jailbreak attempts), tool access
hardening with sandboxing and parameter validation, credential protection
(short-lived tokens, hardware binding, JIT access, ABAC), memory safeguards
(session isolation, context integrity validation, retention policies), and metrics
(dwell time and coverage as the highest-leverage measurements).

Part V covers defensive operations: put a frontier model at the front of the alert
queue for automated first-pass triage, run tabletops for five simultaneous incidents
rather than one, map detection coverage against MITRE ATT&CK, and establish
emergency change procedures in advance. The guide emphasises keeping humans
on containment and disclosure decisions while automating evidence collection,
enrichment, and documentation.

Throughout, "pro-tip" callouts show how Claude Code implements many of the
recommended controls — sandboxed execution, deny-by-default permissions,
OAuth 2.0 token refresh for MCP, OpenTelemetry tracing, managed
organization-wide policies, and session-scoped credentials.
