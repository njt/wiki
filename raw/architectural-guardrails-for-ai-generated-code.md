---
url: https://www.oreilly.com/radar/architectural-guardrails-for-ai-generated-code/
date_fetched: 2026-09-16
---

Consider a composite of a failure pattern that’s becoming increasingly common on teams that have scaled AI-assisted development past a handful of enthusiasts.

A staff engineer named Priya opens a pull request. The PR is 340 lines and adds an endpoint that writes to the customer table directly, bypassing the internal customer service API. The code is clean. The tests pass. The AI coding agent that wrote it has been the team’s most productive contributor for six months. The reviewer, three months into the team, approves. The PR ships that afternoon.

Two weeks later, during an integration debug, someone notices that customer records written by that endpoint are missing audit-log entries. The audit hooks live in the customer service API. The team banned direct database access two years ago for exactly this reason and wrote an architectural decision record (a versioned markdown document, one of dozens the team has accumulated, that captured what was decided, why, and what was superseded) to memorialize the rule. The engineer who wrote the ADR has since left. Nobody on the current team remembered the decision. The document was sitting in a directory the current workflow never touches.

The endpoint gets rewritten. Audit gaps get backfilled. The team spends most of a sprint on cleanup. Nobody calls this a failure of AI-assisted development. The AI wrote functional code. It just wrote code that violated a decision the team had already made, in a document the AI had no view into.

This is a specific failure mode. It is not a hallucination since the output was grounded, syntactically valid, and idiomatic. It’s not a model-quality problem since a better model on that same prompt wouldn’t necessarily help if the decision remained absent from its context. It’s a memory problem. Not the model-internal sense of context window, but the organizational sense. The ADR was available in the repository. It was never surfaced to the agent, and the reviewer had not read it.

This piece is about naming what would need to exist for that PR to have been caught, or better, never written in the first place.

## The rework signal

Priya’s PR illustrates one source of a broader rework problem. Faros AI, an engineering analytics platform, published a report in 2026 based on telemetry from more than 22,000 developers across 4,000+ teams. AI-code acceptance rates had risen from 20% to 60% between periods of low and high AI adoption, while code churn (i.e., lines deleted within days of being added) had increased 861% over the same interval.

Faros is careful in how it frames the churn number. The increase may include not only rework but productive refactoring, previously unaffordable cleanup, or faster iterative improvement. But the number still exposes a gap between code entering the repository and code that survives there. A gap that has widened, at scale, alongside AI adoption. Anecdotally, engineers at teams running these tools describe returning to code they had already approved to fix issues that were not obvious at review time. Architectural drift, i.e., code that individually looks fine but collectively pulls the codebase away from where the team agreed it should go, is one plausible contributor.

I’ll call the discipline of preventing this failure mode *architectural drift prevention*: keeping generated code aligned with the architectural decisions a team has already made.

The pattern-matching explanation for that gap has been that the AI isn’t good enough at writing code yet, and a better model will close it. Watch enough review cycles at a team running AI-assisted development for a year, though, and a different pattern emerges. The generated code isn’t obviously bad. It compiles. It passes tests. What it doesn’t do is respect decisions the team has recorded but the AI has never seen.

## Why current tools sit at the wrong layer

Teams commonly reach for several existing mechanisms to close this gap. Most of them are the wrong shape for the problem.

The closest attempts are files like Cursor Rules and CLAUDE.md, i.e., free-text markdown documents dropped into the project root so agents read them as standing instructions. These are the right instinct at the wrong resolution. Free text has no precedence rules, no versioning, no lifecycle. When one rule contradicts another, nothing arbitrates. When a rule is violated, nothing catches it. These are documents in the shape of configuration.

Linters and code formatters operate a layer below. They enforce that a function has a return type annotation or that a variable name follows a convention. They can’t enforce that customer-data writes must go through the customer service API, because that’s not a syntactic property. It’s a semantic decision recorded in a document the linter has no reason to read.

Dependency scanners, SCA tools, and lockfile audits close a related gap; they catch known vulnerable libraries, license violations, and outdated versions. They would’ve flagged nothing about Priya’s PR. Every dependency in it was current, had no known vulnerabilities, and was approved. The violation was a routing choice inside the team’s own architecture, not a library problem.

LLM-assisted code review is the fashionable answer. A second AI reads the pull request and comments on it. This catches surface issues like a swallowed exception, an off-by-one bug. It doesn’t catch drift, however, because the second AI has the same problem as the first: no durable access to the team’s recorded decisions. Two probabilistic passes over the same blind spot are not one deterministic pass with sight.

Human review is the last line, and it works when the reviewer knows the history and has enough time to inspect the change. Agentic development changes both conditions. Agents can produce multiple implementations, pull requests, and revisions in the time it takes a human reviewer to assess one. Code output scales; review attention does not. Asking humans to compensate by reviewing more and harder simply moves the constraint downstream.

**None of these tools are bad. They’re just the wrong layer for the drift problem.**

## What the missing layer would need to do

The missing layer connects recorded engineering decisions to the tools that generate, review, and merge code. Its job is to make the team’s accumulated architectural decisions machine-readable, injectable, and enforceable. I’ll call this layer engineering governance, borrowing the term from adjacent categories like data governance and security governance, where it means the same thing: A structured way for an organization to make explicit and enforceable the rules it already implicitly follows.

At the shape level, the layer needs to do four things.

- It needs to hold decisions in a structured corpus with precedence and lifecycle metadata so that a tool knows which ADR wins when two conflict, and which decisions are still active.
- It needs to retrieve from that corpus reliably, so the same code produces the same set of relevant decisions each time.
- It needs to inject those decisions into the AI’s context before the agent writes code, so the output accounts for them rather than needing to be caught afterwards.
- It needs to enforce them in continuous integration, blocking or flagging code that violates them, with the verdict traceable to a specific ADR, a specific term that matched, a specific rule.

The critical property that ties those four together is a discipline about where probabilistic reasoning is allowed. Probabilistic systems may retrieve or recommend. They shouldn’t independently determine an enforcement verdict. Every block or warning has to reconstruct from artifacts on disk. That’s the code, the ADR, the retrieval log, the rule text. Then, when a developer asks “why did this fail?” or an auditor asks “on what basis?” the answer isn’t “the AI said so.” That’s what makes the layer defensible in the situations where defensibility matters: regulated environments, compliance review, incident retrospectives, and the everyday conversation where an engineer has to justify a blocked merge to the person whose code was blocked.

AI may help surface relevant decisions, but it shouldn’t be the final authority. The enforcement path must remain deterministic: every verdict should resolve to explicit rules, observable evidence in the code, and a result that another person can reproduce.

## Defining the boundaries

Naming a category clearly requires naming what falls outside it. The engineering governance layer, done right, **is not** any of the following:

- **An agent**. Nothing autonomous. Nothing iteratively deciding what to do next. The layer runs when called, produces a verdict, and stops.
- **Memory in the retrieval-augmented-generation sense**. Retrieval methods can vary and may be probabilistic; the enforcement verdict cannot. The corpus of decisions is the source of truth.
- **Code reviewed by AI**. Reviewing generated code with a second model doesn’t address the underlying problem, which is that the first model had no access to the team’s decisions.
- **Vendor-locked**. Production teams increasingly run Cursor, Claude Code, GitHub Copilot, and Codex in parallel, sometimes on the same repository. They may also use open-weight or self-hosted models for sensitive codebases and internal workflows. Engineering governance must remain independent of both the coding tool and the underlying model, so the same architectural decisions and deterministic enforcement apply across them all.

## The emerging engineering stack

The AI coding stack is assembling itself in the open, without anyone architecting it. Each of the major coding assistants is specializing on a different piece of the loop; editing, autonomous execution, review, planning. The layer this stack doesn’t yet have is engineering governance.

The need isn’t limited to regulated industries. Any team that values reliable engineering, long-term product quality, and the trust of its customers needs to know that generated code respects the decisions the system depends on. In higher-risk environments, that requirement becomes formal and auditable. Elsewhere, it’s simply part of building software responsibly. In both cases, the enforcement path should be deterministic and traceable, with the rules, evidence, and verdict open to inspection rather than hidden inside another model or a proprietary black box.

## For engineering leaders reading this today

Priya’s PR is a pattern, not an incident. Three things worth doing this quarter, regardless of tooling choices.

- **Audit your architectural decision records.**Are they current? Do they explicitly name the decisions they replace? If your team doesn’t write ADRs, this is the moment to start. The tooling that will exist in twelve months assumes structured architectural decisions as input.
- **Choose the enforcement posture deliberately**. The system can either warn the developer and let them continue or block the change until the issue is resolved. Both approaches can work, but the team should agree on which applies instead of leaving the decision to each developer or each pull request.
- **Don’t assume a more powerful model will solve this problem**. Better models can improve the code they generate, but they still can’t follow architectural decisions they’ve not been given. Preventing architectural drift requires changing the surrounding system, not simply waiting for the next model.

The productivity gains from AI-assisted development are real and worth having. So is the architectural coherence teams spent years building. Engineering governance is the layer that lets you keep both.
