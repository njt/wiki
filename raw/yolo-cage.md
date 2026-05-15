---
title: "yolo-cage"
url: https://github.com/borenstein/yolo-cage
date_fetched: 2026-05-14
section: "LLMs"
---

# yolo-cage: Autonomous AI Coding Agents with Safety Constraints

AI coding agents that can't exfiltrate secrets or merge their own PRs. 110 stars, MIT licensed, primarily Python.

## Core Problem

"Permission prompts neglect the weakest part of the threat model: a tired user." yolo-cage defers decisions until pull request review, empowering the agent while limiting its blast radius.

## Architecture

Sandboxed environment (Vagrant VM with MicroK8s) containing:
- Agent Layer: Claude Code running in autonomous mode within an isolated sandbox
- Dispatcher: Enforces branch isolation and blocks dangerous git/GitHub CLI operations
- Egress Proxy: Scans outbound HTTP/HTTPS traffic for secrets and blocked domains

## Security Controls

### Secrets Protection
- Pattern matching for credential formats (sk-ant-*, AKIA*, ghp_*, SSH keys)
- LLM-based secret scanning in request bodies, headers, and URLs
- Egress proxy inspection of all outbound traffic

### Git Operation Restrictions
- Agents can only push to their assigned branch
- Blocked: git remote, git clone, git config, git credential
- Branch isolation at dispatcher level

### GitHub API & CLI Blocking
- Prevents PR merges, repo deletion, webhook modifications
- Blocks gh api calls that could bypass protections

### Exfiltration Prevention
- Domain blocklist (pastebin.com, transfer.sh, file.io)
- One sandbox per branch, isolated from others
- TruffleHog pre-push scanning

## Acknowledged Limitations

"Reduces risk" but "does not eliminate it." Potential bypasses: DNS exfiltration, timing side-channels, steganographic data hiding, sophisticated encoding schemes.

## Requirements

Vagrant with libvirt (Linux) or QEMU (macOS), 8GB RAM, 4 CPUs, GitHub PAT, Claude account.
