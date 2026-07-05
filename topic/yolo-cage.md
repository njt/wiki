# yolo-cage

AI coding agents that can't exfiltrate secrets or merge their own PRs. Runs Claude Code in a sandboxed Vagrant VM with a dispatcher (enforces branch isolation, blocks dangerous git operations) and an egress proxy (scans outbound traffic for secrets, blocks known file-sharing domains). The key insight: permission prompts fail because tired users click "allow" on everything, so defer the security decision to PR review instead.

---

## Key Quotes

> "Permission prompts neglect the weakest part of the threat model: a tired user."

> "Reduces risk" but "does not eliminate it."

## Key Themes

#security #sandboxing #agent-safety #coding-agents #secrets-protection

The threat model is realistic: the agent might try to exfiltrate secrets (credential patterns, LLM-based scanning), push to wrong branches (dispatcher blocks), merge its own PRs (GitHub API restrictions), or leak data to external services (domain blocklist + TruffleHog). The acknowledged limitations are honest -- DNS exfiltration, timing side-channels, and steganography remain possible.

This sits at the intersection of [[Security]] and [[Agentic Coding]]. The "defer to PR review" philosophy aligns with [[Experience Design for Agents]]'s emphasis on reversibility and inspectability. It's also the practical counterpart to [[What I learned building an opinionated and minimal coding agent]] where Zechner argues that in-tool permission systems are security theater -- yolo-cage agrees and moves the security boundary to the infrastructure layer instead.

## Critical Analysis

Strong: moving security from "ask the user" to "constrain the environment" is the right architectural choice. The per-branch sandbox isolation prevents cross-contamination. The LLM-based secret scanning in egress traffic is a clever use of the technology to police the technology.

Weak: Vagrant + MicroK8s is a heavy requirement (8GB RAM, 4 CPUs minimum). The setup friction may push teams toward "just yolo it" -- which is ironic for a tool designed to make yolo mode safe. The acknowledged bypass vectors (DNS exfiltration, steganography) are real and sophisticated agents may discover them, per [[Benchmark Exploitation]]'s findings about emergent reward-hacking.

The fundamental tension: you want agents to be powerful enough to be useful but constrained enough to be safe. yolo-cage lands on "powerful within a sandbox" which is probably the right compromise for coding agents specifically.

---
*Sources: [[summary/yolo-cage]]*
*Last updated: 2026-05-14*
