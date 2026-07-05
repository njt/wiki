# A Deep Dive on Agent Sandboxes

Pierce Freeman reverse-engineers how Codex CLI sandboxes agent execution at the OS level, walking through the macOS Seatbelt and Linux Landlock+seccomp implementations in detail. The article is the best public documentation of how a production coding agent actually constrains bash execution — not through prompt instructions (which [[claude-ctrl]] correctly calls "not a constraint") but through syscall-level isolation. The key insight: sandboxing must be *default, not optional*, and the gap between "agent that asks permission for everything" and "agent that can work autonomously" is entirely a sandboxing problem.

---

## Key Quotes

> "You probably wouldn't give your new intern access to the prod credentials. But an arbitrary bash session could certainly provide that permissions escalation."

The framing that matters: bash is an escalation primitive. Not because it's malicious, but because it's expressive. Every tool that gives an agent a shell is implicitly giving it everything the user can access.

> "By default, you're sandboxed."

The entire design philosophy in four words. Contrast with Claude Code's command-whitelist approach, which Freeman calls "brittle and a bit annoying" and "fully impractical to run if you step away from your computer." Default-deny at the OS level vs. default-allow with human approval — these are different threat models, not just different UX choices.

> "The bug I reported (in an AI agent sandbox using Seatbelt) was that it failed to properly block off access to the user's home directory dotfiles."

The recursive problem: sandbox policies are code, and code has bugs. Custom Seatbelt policies are particularly error-prone because they use a obscure Scheme-like DSL that few developers understand. The sandbox itself needs a sandbox.

> "This kind of sandboxing is going to be pretty important" as an alternative to pleading with the agent not to delete your home directory.

Freeman's understated conclusion. The current landscape is split between OS-level isolation (Codex, [[Zeroclaw]]), container isolation ([[yolo-cage]], [[OpenSandbox]], [[Navaris]]), and prompt-level pleading (everyone else). The first two are engineering; the third is hope.

## Key Themes

#sandboxing #security #agents #coding-agents #codex #seatbelt #landlock #seccomp

**The execution pipeline.** Every tool call routes through `process_exec_tool_call` based on `SandboxType`. On macOS, commands spawn under `sandbox-exec` with a generated Seatbelt policy. On Linux, a separate `codex-linux-sandbox` subprocess applies Landlock filesystem rules and seccomp syscall filters before `execvp`-ing the actual command. The policy abstraction (`SandboxPolicy`) is the same regardless of platform.

**Platform divergence is structural, not incidental.** macOS Seatbelt gives coarse-grained control (writable roots, binary network on/off). Linux Landlock+seccomp is more granular — you can allow `recvfrom` (needed for `cargo clippy`'s socketpair-based subprocess management) while denying `sendto`, `connect`, `bind`, and other network syscalls individually. This isn't an implementation detail; it means Linux-sandboxed agents can do strictly more than macOS-sandboxed ones.

**Environment variable hygiene.** `spawn_child_async` wipes the entire environment and rebuilds it from a whitelist. This is the detail most frameworks miss — agents inherit the user's shell environment by default, which means they get API keys, cloud credentials, SSH agent sockets, and everything else. Codex kills it all and starts fresh. [[OneCLI]] and [[You Dont Want Long-Lived Keys]] address the same problem from the credential side, but environment clearing is the execution-side counterpart.

**Session-scoped trust lists.** Commands run through `assess_command_safety`, which categorizes them as safe-to-auto-run, needs-approval, or needs-unsandboxed execution. Approved commands are remembered for the session. Failed sandboxed commands can be retried unsandboxed with user consent. This is a pragmatic compromise between the "approve every `ls`" nightmare and full YOLO.

**The recursive sandbox problem.** Sandbox policies are written by humans, and humans make mistakes — Freeman cites a Seatbelt bug that failed to block home directory dotfiles. The policies themselves need auditing, testing, and verification. `codex debug seatbelt` and `codex debug landlock` are steps in the right direction, but the tooling is still primitive compared to what we have for network security (nmap, Wireshark, etc.).

## Critical Analysis

This is the best single public document on how a production coding agent does OS-level sandboxing. The code excerpts are well-chosen and the architecture walkthrough is clear. But it's narrow in ways that matter.

**Strengths.** The article correctly identifies that sandboxing is the foundation, not an add-on. The platform comparison is substantive — Freeman actually reads the code and explains *why* Linux is more granular. The environment variable clearing detail is the kind of thing that's invisible until it burns you, and calling it out is a service.

**Weaknesses.** The article is exclusively about Codex's approach and doesn't engage with alternatives. Container-based sandboxing (Docker, Firecracker, gVisor) is mentioned only in passing, despite being how most production agent systems actually work ([[OpenSandbox]], [[Navaris]], [[Crabbox]]). The "all-or-nothing network control" limitation on both platforms is noted but not grappled with — how *do* you allow `npm install` but block `curl evil.com`? The answer isn't at the OS level, which means real-world sandboxes need application-layer proxies that don't exist yet.

**The unsolved problem.** The fundamental tension is between utility and isolation. An agent that can't access the network can't install packages, can't call APIs, can't push code. An agent that *can* access the network can exfiltrate data. The session-scoped trust list is a human-judgment escape hatch, which means the human is still the weakest link — exactly the problem [[yolo-cage]] identifies with permission prompts. The architecture is sound, but the trust model still bottoms out in human vigilance.

**What's missing from the analysis.** No discussion of inter-sandbox communication (what if Agent A needs to share a result with Agent B?), no audit trail or forensics tooling (what did the agent do while you were at lunch?), and no security evaluation methodology (how do you test whether the sandbox actually works against an adversarial prompt?). These are [[Security and Sandboxing]]'s open questions, and this article doesn't close them.

**The verdict.** Codex's implementation is "quite nice," as Freeman says, and the design principles are correct. But OS-level sandboxing alone isn't enough for production agent deployments. You need defense in depth: OS isolation as the outer boundary, application-layer validation ([[VTcode]]'s tree-sitter command parsing), credential isolation ([[OneCLI]]), network egress filtering ([[yolo-cage]]'s TruffleHog proxy), and human review at the merge boundary. Codex's sandbox is one essential layer. It's not the whole answer.

---
*Sources: [[summary/a-deep-dive-on-agent-sandboxes]]*
*Last updated: 2026-05-15*
