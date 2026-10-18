# Open Policy Agent

Open Policy Agent (OPA) is a CNCF general-purpose policy engine that evaluates declarative Rego policies against JSON input to make allow/deny decisions — historically for Kubernetes admission, Envoy, API gateways and CI/CD, and now pitched explicitly at governing which tools an AI agent may call and with which parameters. Its homepage's agent-tool-call example is a compact demonstration of the wiki's central guardrail claim: constraints belong in a deterministic system, not in prose the agent is trusted to obey.

---

The pitch is three props, all about *where policy lives*: decoupling policy from application logic so security and platform teams manage shared rules centrally while product teams extend them; performance, by operating on pre-loaded in-memory data so OPA is a "fast policy decision point"; and audit, with every decision recorded and replayable for compliance or debugging. Rego is a domain-specific language, which is the interesting bet — it trades the universality of a general-purpose language for policies that are short, analysable, and diff-able in review.

The examples escalate: payment authorization (account state + risk score + amount), owner-only record updates, Envoy HTTP authorization, Kubernetes admission control requiring a `costcenter-` label prefix. Then the AI example:

> OPA lets you enforce fine-grained policies over which tools an AI agent can call, what parameters are permitted, and how those tools can be used.

The policy denies `Bash`, `Write` and `Edit` outright, restricts `WebFetch` to HTTPS URLs, caps `WebSearch` results at ten, and bounds tool timeouts at 10 seconds — with `deny contains msg` rules producing human-readable failure strings rather than a bare boolean.

## Analysis

This is the strongest possible confirmation of what [[Five Mistakes People Make with Agentic]] called the sharpest mistake in agent configuration: a sentence in AGENTS.md enforces nothing. OPA's agent example is the counter-architecture — the exact same constraints (no `Bash`, HTTPS-only fetches) expressed as code that *cannot* be talked out of, evaluated before the tool call executes. Note the convergence: the constraints any careful practitioner writes in a CLAUDE.md are precisely the ones OPA encodes, but with a replayable audit trail attached to each decision.

The audit prop deserves more attention than the homepage gives it. The wiki's observability sources keep arriving at the same requirement — a per-decision record of what was asked, what was allowed, why — and OPA's decision log gives it for free as a side effect of the enforcement mechanism. There's a nuance worth flagging, though: a deny rule is a *blunt* instrument. Denying `Bash` entirely (the example's `_disallowed_tools`) throws away the productive middle ground between "no shell" and "any shell command" that graded permission systems like Claude Code's sandbox profiles handle more delicately. Rego is expressive enough for graded rules, but the example chooses the fence-post version.

There is also a quiet architectural claim in "pre-loaded, in-memory data": OPA makes policy fast by making it *local and stateless*, which is why it fits agent harnesses — you can evaluate tool-call policy inside the loop without a network hop per tool call.

## Relation to the wiki

OPA strengthens [[A Convention Is Not a Constraint]] with a shipped, CNCF-scale implementation of exactly its thesis — the constraint that binds is the one evaluated by the system, and Rego is what "in the system" looks like when you don't want to hand-roll it. It nuances [[Fences, not Sandboxes]]: the deny-rules example is a fence around each tool call rather than a sandbox around the process, suggesting the two are complements (fence what the agent may request, sandbox what it executes). And it gives [[Zero Trust for AI Agents]] a concrete enforcement primitive — every tool call treated as untrusted input to a policy decision point, with the decision log as the zero-trust audit trail. The blunt `Bash` deny also sharpens the argument in [[Five Mistakes People Make with Agentic]], while showing that even the deterministic answer has a coarseness worth pushing against.

#tool #concept

---

*Sources: [[raw/openpolicyagent-org]], [[summary/openpolicyagent-org]]*
*Last updated: 2026-10-18*
