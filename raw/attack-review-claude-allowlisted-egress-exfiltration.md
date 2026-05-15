---
title: "Attack Review: Claude Allowlisted-Egress Exfiltration"
url: https://gist.github.com/lhl/7c6d7085185931978ad6ed2d2de72081
author: lhl
date_fetched: 2026-05-15
date_published: 2026-04-07
---

# Attack Review: Claude Allowlisted-Egress Exfiltration

Two independent researchers discovered the same root-cause vulnerability: the Anthropic API is implicitly allowlisted in Claude's sandbox, and attackers can co-opt it as an exfiltration channel using their own API keys. The gist analyzes the attack through the lens of a security framework called **shisad**, contrasting Claude's vulnerabilities with shisad's defenses across four kill chain stages.

## Attack Summary

Both attacks follow a 4-stage kill chain: injection delivery (hidden instructions in a document), data harvesting (reading local files or chat history), exfiltration (uploading data via the Files API to the attacker's Anthropic account), and no approval gate (the chain executes without human confirmation).

### Claude Pirate (wunderwuzzi / Embrace The Red, 2025-10-28)

Indirect prompt injection via a malicious document triggers Claude's Code Interpreter. It harvests previous chat conversations through Claude's memory feature, writes data to the sandbox filesystem, then executes a Python files upload call using an attacker-controlled API key. The payload is "obfuscated alongside benign code ('Hello, world' print)." Up to 30 MB per upload with multiple files transferable sequentially. A HackerOne report was "initially closed (2025-10-25) as out-of-scope" then re-opened after pushback.

### Claude Cowork (PromptArmor, 2026-04-06)

A victim connects Cowork to a local folder with confidential files including financial data and partial SSNs. A .docx file contains hidden injection using 1-point font, white-on-white text, disguised as a Markdown "Skill." The injection directs Cowork to use `curl` to send confidential files to the attacker's Anthropic account via the file upload API. "At no point in this process is human approval required." Demonstrated against Claude Haiku; Claude Opus 4.5 also susceptible. A secondary finding: malformed file types can cause persistent API errors in subsequent chats.

## Root Cause

The sandbox allowlists `api.anthropic.com` for its own operations, creating a "trusted egress channel that any code running in the sandbox can reach." An attacker substitutes their own API key, "turning the provider's own infrastructure into an exfiltration endpoint." The "Package managers only" network setting is misleading because the Anthropic API endpoint is always reachable regardless.

## shisad Defenses by Kill Chain Stage

### Stage 1: Injection Delivery
- Pattern-Based Injection Classifier (13 YARA rule files, risk score 0.0–1.0)
- PromptGuard 2 Semantic Classifier (local ONNX inference with tiered thresholds)
- Multi-Encoding Injection Hardening (URL-percent, HTML entity, ROT13 decode passes)
- Unicode Text Normalization (zero-width character stripping)
- Taint Labeling at Ingress (document content labeled UNTRUSTED)
- Evidence Reference Wrapping (HMAC-authenticated stubs)

### Stage 2: Data Harvesting
- Filesystem mount allowlist/denylist with per-tool mount policy
- 8-layer PEP pipeline (tool registry → allowlist → schema validation → capability check → DLP scan → resource authorization → egress check → taint-sink enforcement)
- Resource authorization with object-level ACLs and cross-workspace denial
- Behavioral sequence analyzer detecting "read-then-exfiltrate" pattern
- Memory retrieval taint propagation

### Stage 3: Exfiltration via Allowlisted API Endpoint
- Egress proxy routing ALL outbound HTTP (no direct socket access)
- Deny-by-default egress with no implicit allowlisted domains
- Egress wildcard hardening
- Provenance-aware egress decisions (5-tier model; unattributed contexts → block)
- Credential injection at egress boundary (LLM never sees raw secrets)
- Tool argument DLP scanning for API keys/tokens
- Connect-path network restriction with iptables OUTPUT rules
- DNS rebinding check

### Stage 4: No Approval Gate
- Risk scoring with configurable thresholds (auto-approve / require-confirmation / block)
- One-action confirmation semantics (no batch approvals, replay-safe nonce, 3s cooldown)
- 5-voter consensus system where any single voter can veto HIGH/CRITICAL actions
- Plan commitment protocol (structural plans committed before fetching untrusted content)
- Multi-factor approval for high-risk actions (TOTP/WebAuthn/hardware signer)

## Architectural Takeaway

The fundamental difference is the egress trust model. Claude assumes the provider API is friendly infrastructure, creating a "privileged network path that attacker-controlled code can hijack by supplying its own credentials." shisad treats all egress as untrusted by default with no provider-privileged network path. The model provider's API runs in the daemon process (outside the sandbox), not in tool execution code (inside the sandbox). This "is not a feature-level fix" but "an architectural decision: the sandbox's network surface is empty by default, and every addition is explicit, scoped, and auditable."

## Open Questions

1. Slow exfiltration — whether low-and-slow data encoding into benign outputs over multiple turns could evade detection
2. User-intended adversarial files — the UX tension when a user explicitly requests a file that contains injection
3. Attacker-supplied API keys — obfuscation of keys (splitting across variables, base64 encoding), though deny-by-default egress is the real defense
4. DoS via malformed files — whether crafted input could corrupt subsequent session state

## Sources

- Claude Pirate by wunderwuzzi / Embrace The Red (2025-10-28)
- Claude Cowork File Exfiltration by PromptArmor (2026-04-07)
- HackerNews discussion of the Cowork finding
- Original Claude.ai vulnerability by Johann Rehberger (2025), "acknowledged but unresolved by Anthropic before Cowork release"
