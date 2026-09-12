---
url: https://ruuda.nl/2026/deptool
title: Building the deployment tool I wish I had
author: Ruud van Asseldonk
date_fetched: 2026-05-18
date_published: 2026-05-06
topics:
  - developer-tools
---

# Building the deployment tool I wish I had

By Ruud van Asseldonk, 6 May 2026.

## Opening

The author describes being up at 00:43, pressing `y` to deploy, watching "s4.ruuda.nl: connecting …" flash, then "Applying" appearing briefly before "done." The deploy succeeded.

Ruud acknowledges colleagues might accuse him of "not-invented-here syndrome," reframing it as "having higher standards." The immediate trigger was wanting to write about European digital sovereignty while hosting on US infrastructure — leading to a plan to move hosting to Europe. This cascaded: new VM, DNS changes, self-hosting DNS servers, needing multiple servers, and realizing their simple Python deploy script wouldn't scale.

Friends mentioned: Arian (arianvp) — whispers "Just use NixOS"; David (davidv.dev) — built Unsible.

## The Result: Deptool

One month later, Deptool exists. A deployment tool with a Git-based approach to configuration management.

Example deploy output:
```
$ deptool deploy
s4.ruuda.nl
    update nsd
        ~ zones/ruuda.nl.zone
        restart unit nsd.service

s5.ruuda.nl
    update nsd
        ~ zones/ruuda.nl.zone
        restart unit nsd.service

Auto-rollback if deploy fails.
Apply to 2 hosts in cluster 'prod'? [y/N/d] y

  s4.ruuda.nl: done
  s5.ruuda.nl: done

Changes deployed successfully to 2 hosts in 0.99s.
```

## Wishlist (5 criteria)

1. **Fast** — sub-second config updates
2. **Predictable** — separate plan/apply phases like OpenTofu, not Ansible's unreliable check mode
3. **Safe** — automatic rollback in milliseconds if config breaks
4. **Simple** — copy files, restart systemd units; no control flow or arbitrary code execution
5. **Declarative** — removing a file from config removes it from the server
6. **Zero-setup** — no agents, daemons, enrollment, or registration needed on fresh hosts

## Core Design: Decouple Distribution from Generation

The key idea: separate configuration *generation* from *distribution*. Build config externally, ship it to hosts, keep it simple. Influenced by Nix's approach of storing artifacts where versions coexist, with minimal imperative steps for activation via symlink swaps.

## How Deptool Works (6 mechanisms)

1. **Pre-render configs cluster-wide** into a two-level directory tree (host → application)
2. **Store in Git** — diff across versions, see which hosts/apps changed, get precise file diffs
3. **Materialize in isolated directories** on hosts under `/var/lib/deptool`, named by commit; use a `current` symlink for atomic version swaps
4. **Track deployed state with remote-tracking refs** per host (not cluster-wide), enabling offline diff computation in milliseconds
5. **Lock-then-deploy** — SSH in, request lock with expected commit ref. Lock confirms the plan is still valid. If ref is stale, abort and retry
6. **Restart systemd units** on config change; if unit fails, symlink back to previous version and restart — automatic rollback in milliseconds

## Optimistic Concurrency Model

Same model as `git push`: fast with no contention (single user), performs poorly under heavy contention (teams). "I'm the only person managing my personal infra."

## Building the Agent (for Flatcar Linux)

The server runs Flatcar Linux — image-based, minimal userspace (coreutils + Bash, no Python, no package manager).

Requirements: must work on a fresh host out of the box, use SSH with passwordless sudo.

Why not shell-over-SSH? "the argv also doesn't cross the SSH boundary unscathed" — escaping issues and word splitting make it unreliable.

The solution: Run a single command — a static binary agent at a predictable location. It reads from stdin, writes to stdout. SSH is used only for transport (a socket).

Agent deployment strategy:
1. Build a static binary (no interpreter, no dependency assumptions beyond the kernel)
2. Place it at a commit-named path: `/var/lib/deptool/bin/deptool-<version>-<commit>`
3. Optimistically assume the binary is already there (common case — config changes more often than tool updates)
4. On failure, install via a second SSH connection using coreutils commands: `uname -sm` to detect OS/arch, `dd` to write binary to disk, `sha256sum` to verify transfer

The binary is ~1.6 MB. SSH ControlMaster keeps connections warm, making subsequent deploys feel instant.

"All user-controlled input goes over our SSH-backed socket, it never enters SSH or shell commands."

## Conclusion

Ruud has been using Deptool for a month and is "very pleased with it." Sub-second deploys are the standout feature — "it's *faster* to make the edit locally and deploy, than to SSH into the server."

"Every applied edit gets recorded in the Git history, and if I break something, Deptool rolls back before I even realize it was broken."

## Links

- Deptool manual: https://docs.ruuda.nl/deptool/
- Codeberg: https://codeberg.org/ruuda/deptool
- GitHub: https://github.com/ruuda/deptool
- Flatcar Linux: https://www.flatcar.org/
- Unsible: https://github.com/ChorusOne/unsible
- RCL templating docs: https://docs.ruuda.nl/rcl/generating_files/
