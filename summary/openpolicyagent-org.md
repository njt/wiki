---
url: https://www.openpolicyagent.org/
title: "Open Policy Agent — Policy-based control for cloud native environments"
author: "Open Policy Agent maintainers (CNCF project)"
date_fetched: 2026-10-18
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - security-and-sandboxing
---

Open Policy Agent (OPA) is a CNCF-hosted, general-purpose policy engine: policies are written in Rego, a declarative domain-specific language, and evaluated against JSON input to produce allow/deny decisions. The homepage pitches three value props — decoupling policy from application logic so security and platform teams own centrally managed shared policies, speed (pre-loaded in-memory data, "a fast policy decision point"), and audit (every decision is recorded and replayable for compliance or debugging).

The page walks through progressively richer Rego examples: a payments authorization rule (account state, risk score, transaction amount), application-level authz (only the owner may update a pet record), Envoy external authorization over HTTP headers and paths, Kubernetes admission control enforcing costcenter label conventions, and — the part most relevant to this wiki — fine-grained control over which tools an AI agent may call, with which parameters:

```rego
deny contains $"Tool {tc.tool} is not allowed" if {
    some tc in input.tool_calls
    tc.tool in _disallowed_tools
}
```

That example denies `Bash`/`Write`/`Edit` outright, restricts `WebFetch` to HTTPS URLs, caps `WebSearch` result counts, and bounds tool timeouts — the same shape of constraint the wiki's agent-control sources argue for, expressed as a deterministic policy rather than prose instructions. OPA integrates via SDKs or REST API and is positioned for enforcement "in applications, proxies, Kubernetes, CI/CD pipelines, API gateways, and more."
