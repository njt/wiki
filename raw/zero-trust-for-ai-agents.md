---
title: "Zero Trust for AI Agents"
url: https://cdn.prod.website-files.com/6889473510b50328dbb70ae6/6a1611a04085d7cd3dadc924_Claude-eBook-Zero-Trust-for-AI-Agents-05182026.pdf
author: "Anthropic"
date_fetched: 2026-05-31
date_published: 2026-05-18
---

# Zero Trust for AI Agents

A security framework for deploying autonomous AI agents in the enterprise. Published by Anthropic as a PDF ebook.

## Summary

Anthropic's guide applies Zero Trust principles to agentic AI deployments while addressing current threat vectors. The document is organized into five parts: security considerations for autonomous systems, current threats to agentic systems, applying Zero Trust to agentic AI services (the core framework with three capability tiers), an agent implementation workflow (8 phases), and defensive operations at the speed of autonomous threats.

## Key Framework

### Zero Trust Principles
- Never trust and always verify
- Assume breach
- Least privilege

### The "Impossible vs. Tedious" Design Test
When evaluating any control, ask: does this make the attack impossible, or just tedious? Mitigations whose value comes from friction rather than a hard barrier degrade significantly against an adversary that can grind through tedious steps at scale. Agentic attackers have unlimited patience and near-zero per-attempt cost. Controls that survive: hardware-bound credentials, expiring tokens, cryptographic identity, and network paths that do not exist rather than paths that are merely inconvenient.

### Three Capability Tiers
1. **Foundation** — Minimum viable security. Entry requirements raised by AI-accelerated offense: short-lived tokens, cryptographically rooted identity, identity-based isolation, automated first-pass triage. Friction-only controls no longer qualify.
2. **Enterprise** — Standard practices for organizations with significant deployments. Adds depth for real-world complexity: larger teams, multiple agentic deployments, environments where a single compromise carries meaningful business impact.
3. **Advanced** — Aspirational for most, baseline for high-risk/regulated environments. Hardware-backed identity, continuous authorization, machine learning behavioral analysis, self-healing systems.

### Eight Security Capability Domains (each with tier tables)
1. Agent identity and authentication — Cryptographic identifiers, mTLS, hardware-bound credentials
2. Access control and privilege management — RBAC, ABAC, continuous authorization, JIT/JEA
3. Resource boundaries — Identity-based isolation, sandboxed execution, confidential computing
4. Observability and auditing — Action logging, immutable audit trails, SIEM streaming
5. Behavioral monitoring and response — Baselines, anomaly detection, automated response
6. Input validation and output controls — Sanitization, spotlighting, output filtering, HITL approval
7. Integrity and recovery — Version-controlled configs, signed configs, immutable infrastructure, self-healing
8. AI governance policies — Acceptable use, formal governance frameworks, automated compliance

### Threat Landscape (Part II)
- **Prompt injection**: Direct (user input overrides) and indirect (poisoned external data). Microsoft's Spotlighting technique reduces indirect injection success from >50% to <2%.
- **Tool poisoning**: Malicious MCP tool descriptors, rug pull attacks. First documented in-the-wild malicious MCP server impersonated a legitimate email service and copied all sent emails.
- **Tool chaining attacks**: Combining legitimate tools in harmful sequences — a CRM tool + email tool to exfiltrate data, neither individually suspicious.
- **Unscoped privilege inheritance**: High-privilege manager delegates to worker without scoping. Confused deputy problem amplified by routine agent coordination.
- **Memory-based privilege retention**: Agents cache credentials across sessions, enabling escalation across session boundaries.
- **Supply chain risks**: Poisoned model weights (250 malicious documents can backdoor LLMs 600M-13B params, persisting through RLHF). Malicious Hugging Face models with reverse shells. PyTorch dependency confusion attack.
- **Memory and context poisoning**: RAG poisoning, shared context poisoning, long-term memory drift.

### Eight Implementation Phases (Part IV)
1. Identify requirements — Regulatory, operational, constraints. Align security/legal/compliance/business.
2. Manage supply chain risks — AI-BOM, OpenSSF Scorecard, dependency tree audit, reachability analysis, AI vendoring for unmaintained deps, cryptographic signing, vendor assessments.
3. Define agent boundaries — Approved/prohibited actions, escalation triggers, scope limits (Least Agency), blast radius identification.
4. Defend against prompt injection — Input isolation, constitutional classifiers (blocked 95% of jailbreak attempts), limit attack surfaces, capability restrictions.
5. Secure tool access — Tool allow-listing, parameter validation (PreToolUse hooks), sandbox execution, approval escalation, rate limiting / spending controls.
6. Protect agent credentials — Short-lived tokens (minutes not days), certificate-based identity, hardware-bound credentials, credential isolation, JIT access, ABAC.
7. Safeguard agent memory — Memory isolation, context integrity validation (cryptographic hashes, source attribution), retention policies, rollback to known-good states.
8. Measure what matters — Dwell time, coverage, detection speed, explainability, behavioral conformance.

### Defensive Operations (Part V)
- Put a frontier model at the front of your alert queue for automated first-pass triage
- Move humans off bookkeeping onto decisions: automate evidence collection, enrichment, correlation
- Map detection coverage against MITRE ATT&CK — prioritize lateral movement and credential access
- Run tabletops for five simultaneous incidents, not one
- Establish emergency change procedures in advance
- Apply Zero Trust to defensive agents too — they can be compromised

### New Terminology
- **Least Agency** (OWASP coinage): Extends least privilege to agentic applications — restricts not just what agents can access, but what each tool can do, how often, and where.
- **Blast radius**: Damage potential if an agent is compromised.
- **Dwell time**: Time between anomaly occurrence and human awareness — the single most leverageable metric.
- **AI-BOM**: AI Bill of Materials extending software composition analysis to model provenance, training data lineage, and fine-tuning parameters.

## Claude Code "Pro-tips" Throughout
The document includes Claude Code-specific implementation tips in callout boxes across every section, covering: deny-by-default permissions, sandboxed execution, OpenTelemetry metrics, command injection detection, input sanitization, OAuth 2.0 with automatic token refresh for MCP, OS credential store, ephemeral sub-agents, session isolation, configurable retention policies, checkpoints/rewind, managed settings for organization-wide policy enforcement.

## Framework Sources Cited
- NIST SP 800-207 (Zero Trust Architecture)
- NSA Zero Trust Implementation Guides (ZIGs, 2026)
- OWASP Top 10 for Agentic Applications (2026)
- MITRE ATLAS (AML.T0051.000, AML.T0051.001)
- Microsoft Research: Spotlighting technique for indirect prompt injection
- Anthropic: Sleeper agents research, constitutional classifiers
- OpenSSF Security Scorecards
- Real-world incidents: Invariant Labs MCP GitHub vulnerability, Postmark MCP npm backdoor, PyTorch dependency confusion, malicious Hugging Face models, Zenity Labs AgentFlayer
