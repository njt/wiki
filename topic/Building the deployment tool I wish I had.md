# Building the deployment tool I wish I had

Ruud van Asseldonk's walkthrough of Deptool, a Git-centric deployment tool he built in one month for his personal infrastructure. The core insight: separate config generation from distribution, then make distribution a solved problem with atomic symlink swaps, remote-tracking Git refs, and a static binary agent that needs nothing but SSH and coreutils.

---

## Key Quotes

> "It's *faster* to make the edit locally and deploy, than to SSH into the server."

This is the line that sells the tool. Sub-second deploys change the economics of configuration work — when deployment is cheaper than `ssh`, you stop SSH-ing into boxes entirely.

> "Every applied edit gets recorded in the Git history, and if I break something, Deptool rolls back before I even realize it was broken."

The safety guarantee that makes speed usable. Fast deploys without rollback would be terrifying; rollback without speed would be tedious. The combination is what makes this feel like a genuine advance over Ansible/Chef/Puppet.

> "All user-controlled input goes over our SSH-backed socket, it never enters SSH or shell commands."

A security property won by design, not by review. The static binary agent reads stdin/writes stdout — SSH is purely transport. No escaping, no word splitting, no injection surface.

> "the argv also doesn't cross the SSH boundary unscathed"

The single best one-line explanation of why shell-over-SSH tooling is quietly broken. Most deployment tools paper over this with quoting gymnastics; Deptool sidesteps it entirely.

---

## Key Themes

#tool #deployment #git #simplicity #infrastructure

**Decouple generation from distribution.** Build config externally, ship the artifact. This is the Nix insight applied to deployment: generation is the hard, language-specific part; distribution should be a dumb pipe. Deptool proves you can make distribution fast, safe, and zero-setup if you're willing to constrain the problem.

**Git as deployment state.** Configs live in a Git repo. Each deploy is a commit. Each host has a remote-tracking ref. Diffing deployed-vs-intended is `git diff` — free, offline, millisecond-fast. This is so obviously correct that you wonder why more tools don't do it.

**Atomic symlink swap + auto-rollback.** Config versions coexist on disk under `/var/lib/deptool/<commit>/`. A `current` symlink points to the active version. Deploy swaps the symlink, restarts units. If a unit fails, swap back and restart — milliseconds. This is the deployment equivalent of a database transaction.

**Optimistic concurrency as a feature, not a bug.** The lock-then-deploy model assumes single-user access. Ruud is explicit: "I'm the only person managing my personal infra." This isn't a limitation he hasn't thought about — it's a deliberate scope choice that eliminates entire categories of complexity.

**Static binary agent via coreutils.** The agent distribution strategy is a small masterpiece: ship a 1.6MB static binary, optimistically assume it's already on the host (it usually is), and fall back to `uname -sm` + `dd` + `sha256sum` for initial installation. No Python, no package manager, no daemon. Works on a fresh Flatcar Linux box out of the box.

---

## Critical Analysis

This is the best kind of "I built my own X" post: honest about scope, precise about design decisions, and opinionated without being preachy. Ruud isn't arguing you should use Deptool — he's showing you what's possible when you refuse to accept the complexity of existing tools.

The design is a clinic in constraint-driven engineering. Every decision flows from the constraints: Flatcar Linux (no Python, no package manager), single user (optimistic concurrency), personal infra (no team coordination). When you accept these constraints, the design space collapses to something small and elegant. Most deployment tools try to serve everyone and get big; Deptool serves one person and stays small.

The "just use NixOS" whisper from Arian is the elephant in the room. Ruud acknowledges it and moves on — Deptool isn't competing with NixOS, it's solving a narrower problem. But the philosophical overlap is real: both separate generation from distribution, both use symlink-based atomic swaps, both store artifacts by content hash. Deptool is Nix's deployment layer, extracted and simplified.

What's missing: no discussion of secret management. Where do API keys, TLS certs, and database passwords live? The Git repo presumably doesn't contain secrets, but the article doesn't address how they get to hosts. Also no treatment of drift — what happens when someone SSHes in and edits a file directly? The remote-tracking ref pattern catches config drift but the article doesn't explore the failure mode.

The Flatcar Linux constraint is both a strength and a weakness. The static binary + coreutils fallback is elegant precisely because the target is so minimal. On a general-purpose Linux box with Python, Ruby, and a package manager, the constraint is artificial and the elegance becomes over-engineering. The tool is beautiful *because* the target is spartan.

For teams, the optimistic concurrency model would break. The article admits this. But the admission is the point: not every tool needs to serve teams. The market is full of team-scale deployment tools (Ansible, Chef, Puppet, OpenTofu, Pulumi). What's rare is a tool that's *designed for one person* and makes no apologies for it. That's the actual innovation here — not the technology, but the refusal to design for a use case that doesn't exist.

---

## Cross-Links

- [[Designing a Passively Safe API]] — auto-rollback on unit failure is passive safety applied to deployments
- [[Simplicity in the Age of AI-Assisted]] — Ruud threw away his Python deploy script and rebuilt simpler; this is the demolition argument in practice
- [[Make the Easy Change Hard]] — building the tool *before* doing the migrations is the same inversion: invest in infrastructure, then the work is easy
- [[Verbose Deployment]] — the opposite end of the deployment spectrum: 10-phase pipeline vs. 3-phase deploy. Both valid, different constraints
- [[Correct by Construction]] — atomic symlink swaps are correctness by construction for deployments
- [[Elements of Code]] — Deptool exemplifies "wrong in correctable ways": when something breaks, the rollback mechanism is the correction
- [[Software Engineering Craft]] — this is craft: recognizing that a Python script won't scale, building the right abstraction, shipping it
- [[Headscale]] — self-hosted infrastructure tooling; same spirit of running your own stack
- [[Microservices for the Benefits, Not the Hustle]] — Deptool is microservice-scale tooling for personal infra: the right amount of architecture for the problem

---

*Sources: [[summary/ruuda-deptool]]*
*Last updated: 2026-05-18*
