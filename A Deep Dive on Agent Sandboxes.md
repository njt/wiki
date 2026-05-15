# A Deep Dive on Agent Sandboxes

Pierce Freeman reverse-engineers how Codex sandboxes agent execution, examining the OS-level primitives (macOS Seatbelt, Linux Landlock + seccomp) that make unattended coding agents safe enough to run. This matters because the gap between "agent that asks permission for everything" and "agent that can actually work autonomously" is entirely a sandboxing problem.

---

## Key Quotes

> "You probably wouldn't give your new intern access to the prod credentials. But an arbitrary bash session could certainly provide that permissions escalation."

## Key Themes

#sandboxing #security #agents #coding-agents #infrastructure

The three permission levels (Read Only, Auto, Full Access) map a clean design space. The "Auto" default -- workspace edits allowed, network blocked -- is the sweet spot for most coding work. It's the same trade-off that [[VTcode]] makes with its multi-layered defense (tree-sitter validation, OS-native sandboxing, configurable human-in-the-loop).

The platform divergence is telling: macOS Seatbelt gives you coarse-grained control, while Linux Landlock + seccomp is more granular but more complex. The all-or-nothing network control on both platforms is a real limitation -- you can't say "allow npm registry but block everything else" at the OS level.

The design principle that sandboxing is *default, not optional* is important. Most agent frameworks treat security as an add-on. Codex treats it as the foundation.

## Critical Analysis

This is an excellent technical teardown, but it's focused narrowly on Codex's approach. The bigger question -- how do you sandbox agents that need to interact with external services, APIs, and databases? -- remains largely unanswered. The "session-scoped trust lists" are a pragmatic compromise, but they still require human judgment about what to trust.

The environment variable clearing is a smart detail that most frameworks miss. Agents inherit the user's shell environment by default, which means they get API keys, cloud credentials, and everything else. Codex wipes the slate and rebuilds selectively.

Missing from the analysis: container-based sandboxing (Docker, Firecracker), which is how many production agent systems actually work. The OS-level primitives described here are elegant for local development but don't address server-side agent execution.

---
*Sources: [[raw/a-deep-dive-on-agent-sandboxes]]*
*Last updated: 2026-05-14*
