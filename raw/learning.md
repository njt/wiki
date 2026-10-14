---
url: https://asibackbone.github.io/Learning/
date_fetched: 2026-10-14
---

# AsiBackbone Learning

Practical .NET architecture for accountable systems

Learn governed execution, secure application design, policy-driven systems, and safe AI integration through focused tutorials, hands-on labs, and runnable examples.

Learning makes the reasoning visible: why an operation was permitted, under which policy, with what evidence, and when a simpler design is enough. Use it before you adopt any implementation, or when a .NET system you design or review needs explicit decisions and audit evidence.

A proposed action should become a governed decision before it becomes real-world execution.

## Start here

**New to the project**Follow the recommended introduction to the core concepts.

**I have a problem to solve**Choose a short, goal-specific route through the material.

**I am evaluating by role**Start from your developer, architecture, platform, AI, or security responsibility.

**The vocabulary is new**Map Learning terms to established architecture concepts.

**I already use ASP.NET Core authorization**See when framework-native authorization is enough—and when the problem is broader.

## Is Learning a fit?

### Use Learning when

- you need to explain why an operation was permitted, under which policy, and with what evidence;
- you are comparing governed execution with familiar .NET patterns before choosing an implementation; or
- you want runnable examples and labs rather than a package to install.

Learning does not require installing an `AsiBackbone` package. Use it as independent .NET architecture education or connect it to the working repositories.

### A simpler design may be enough when

- a role or policy check on the endpoint fully answers who may act: see When ASP.NET Core Authorization Is Enough;
- one service call, one transaction, and ordinary logs describe the whole operation: see When a Simple Application Service Is Enough.

## The core model

The material uses a recurring separation of responsibilities:

- Intent
- Context
- Constraints
- Decision
- Decision receipt
- Acknowledgment when required
- Scoped authority
- Host-owned execution

## Choose a learning path

### Architecture

Explore boundaries, responsibilities, tradeoffs, and structural patterns behind governed systems.

Explore Architecture### Governance

Connect intent, policy context, explicit decisions, acknowledgment, scoped authority, and audit evidence.

Explore Governance### ASP.NET Core

Study secure defaults, middleware organization, operational structure, and modern .NET implementation patterns.

Explore ASP.NET Core### Security

Examine trust boundaries, least authority, explicit control flow, defensive defaults, and accidental privilege.

Explore Security### AI Integration

Apply governed execution to AI-assisted systems, tool calls, agents, workflows, and host-controlled effects.

Explore AI Integration### Tutorials

Move from a familiar implementation through failure modes, tradeoffs, and working references.

Browse Tutorials### Executable Samples

Run small .NET companion samples for the foundation, governance, security, and ASP.NET Core paths, each with focused tests that make architectural invariants observable.

Browse Executable Samples### Labs

Practice with exercises, incomplete implementations, architecture critiques, and policy scenarios.

Browse Labs### Case Studies

See Learning boundaries composed in realistic simulated scenarios without prescribing a production framework.

Browse Reference Architecture Case Studies### Advanced Topics

Explore complex integration patterns, alternative approaches, and deeper architectural questions.

Explore Advanced Topics## How Learning works

Tutorials use a problem-first progression:

- Problem
- Common implementation
- Failure mode
- Architectural pattern
- Teaching example
- Tradeoffs
- Working example

The goal is to make the reasoning visible—not to prove that one framework or architecture is always correct. Study a pattern, reimplement it, compare alternatives, identify when a simpler design is better, and treat working repositories as specimens rather than unquestioned templates.

## Recently added

Learning 1.3 is the current documentation baseline aligned with its reviewed AsiBackbone 7.0.0 boundary. AsiBackbone 7.1.0 is the owning repository's newer stable release; the dated 7.1.0 currentness review records why Learning retains `v7.0.0` for this archival baseline. The compatibility and API boundary pins the selected implementation references; earlier Learning snapshots remain available as historical guidance. Recent publications include:

- An Allowed Decision Does Not Mean the Operation Succeeded
- Idempotency Does Not Prevent Authority Replay
- Should Authorization Fail Open, Fail Closed, or Defer?

New publications are announced in the RSS feed.

## Scope, terminology, and important limitations

Canonical patterns document what the working repositories currently do; alternative patterns create room for comparison, criticism, and improvement. Canonical does not mean universal.

AsiBackbone Learning is an educational architecture resource—not a compliance certification, legal standard, security guarantee, AI model, AGI or ASI implementation, robotics controller, replacement for application-specific security review, or requirement to use the AsiBackbone package. Examples are teaching artifacts; production systems remain responsible for their own security, infrastructure, persistence, regulatory requirements, safety controls, and execution.

`AsiBackbone` is the product name; no acronym expansion is required to follow the material. AsiBackbone Learning is not affiliated with the Artificial Superintelligence Alliance.

## Related projects

### AsiBackbone Learning Current site

Practical .NET architecture tutorials, labs, runnable samples, and reference patterns for governed execution, secure applications, AI integration, and policy-driven systems, including comparisons that show when these patterns are unnecessary.

### AsiBackbone

A .NET governance and policy-control framework for auditable decision flows: policy evaluation, auditable decision records, acknowledgment workflows, capability-based authorization, host-owned execution boundaries, and ASP.NET Core integration.

### .NET Core Application Template

A secure-by-default ASP.NET Core application template with authentication and authorization, structured logging, security headers, rate limiting, EF Core data access, reverse proxy support, and Problem Details error handling.

Treat the working repositories as specimens to study, question, and compare rather than unquestioned templates.

## Design philosophy

We favor architectures where consequential operations are explicit, constrained, reviewable, and auditable.

We also document when these patterns are unnecessary: see When ASP.NET Core Authorization Is Enough and When a Simple Application Service Is Enough.

The projects focus on practical .NET engineering. They do not claim regulatory certification or automatic compliance.

Read it. Run it. Question it. Improve it.
