---
url: https://theupdateframework.io/
title: "The Update Framework (TUF)"
author: The Update Framework project (CNCF)
date_fetched: 2026-10-18
date_published:
topics:
  - security-and-sandboxing
---

The Update Framework (TUF) is a specification and framework for securing software update systems — including package managers, OS updaters, and application auto-update mechanisms. Its distinguishing claim is resilience against attackers who have already compromised the update repository or the signing keys, not merely against network attackers.

The site itself is short: TUF "maintains the security of software update systems, providing protection even against attackers that compromise the repository or signing keys," and offers a flexible specification developers can adopt into any update system. TUF originated from researchers at NYU and is a graduated CNCF project; its threat model underlies real deployments (Docker Notary, Linux distributions, Python's pip ecosystem via PyPI).

The core idea is role separation with multiple signing keys and threshold requirements, plus key-rotation metadata with expiry timestamps — so a single stolen key neither lets an attacker push arbitrary updates nor freezes updates indefinitely by withholding new metadata.
