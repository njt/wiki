---
url: https://www.shankuehn.io/post/ditch-the-token-headache-ssh-just-works
title: "Ditch the Token Headache...SSH Just Works!"
author: Shan Kuehn
date_fetched: 2026-09-16
topics:
  - developer-tools
  - security-and-sandboxing
---

Shan Kuehn makes the case for switching GitHub authentication from Personal Access Tokens to SSH keys, aimed at developers who keep getting bitten by token expiry and per-repo scope bookkeeping. The post opens with the now-standard context: GitHub retired password authentication over HTTPS in August 2021, so the choice is between PATs and SSH — and Kuehn advocates SSH for day-to-day work because it is configure-once.

The bulk of the post is a five-minute setup walkthrough: check for an existing ed25519 keypair, generate one with `ssh-keygen`, load it into `ssh-agent` (with the macOS keychain flag so it survives reboots), copy the public key with `pbcopy`, register it under GitHub's SSH key settings, and verify with `ssh -T`. The key conceptual point is that SSH authentication is keyed to the host (github.com), not to individual repositories — one registered key covers every repo you own, contribute to, or clone, forever.

The only per-repo work remaining is ensuring the remote uses an SSH URL rather than HTTPS, via `git remote set-url` for existing clones or cloning with the SSH URL from the start. Kuehn concedes PATs remain the right tool for CI pipelines and scripts that need scoped, expiring access, but argues that for interactive development SSH "just sits there and works."
