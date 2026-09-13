---
url: https://shiftmag.dev/ai-agents-need-their-own-identity-and-least-privilege-access-11727/
title: "AI agents need their own identity and least-privilege access"
author: ShiftMag interview with Ross Kukulinski (Tailscale)
date_fetched: 2026-09-13
date_published: unknown
topics:
  - security-and-sandboxing
  - mcp-and-tool-protocols
---

A ShiftMag interview with Ross Kukulinski of Tailscale (talk given at WAD Berlin) arguing that infrastructure access should be granted to identities, not to network locations — and that AI agents make this urgent by adding non-human principals that teams habitually over-trust.

The core mismatch: the internet was built to be open, but private infrastructure usually shouldn't be. Teams patch this with firewalls, gateways, proxies and segmentation; Kukulinski's alternative is point-to-point connectivity where policy is governed centrally but enforced at the edge, and where networks stay small and isolated until a resource genuinely needs to be shared. The developer-facing idea is the **separation of authorization from network topology**: a service is reachable because a particular identity is allowed to reach it, not because both endpoints sit inside one trusted network. Identity is more durable than an IP address — pod IPs churn in Kubernetes and subnets prove nothing — and an authenticated connection can carry user, group, device and policy context, which also removes the credential-creation-and-revocation chores around SSH, databases and cluster admin. When access derives from identity-provider groups, a role change updates permissions automatically, with no VPN rules or long-lived keys to clean up.

For AI specifically, Kukulinski warns against the reflex of granting broad access to unblock a project — "give AI access to the whole network" repeats the over-permissive VPN failure mode — and prescribes treating every agent as a separate workload with its own identity, a changeable/revocable access policy, and network isolation down to only the nodes, services and data it needs. He also flags AI-assisted attacker reconnaissance (defenders should apply the same automation to supply-chain security, environment checks and build/deploy controls), an identity-aware gateway pattern for mediating developer access to model providers, and the honest admission that cross-cluster, cross-cloud Kubernetes connectivity remains hard even with identity handled — direct encrypted connections first, relay infrastructure only as backup.
