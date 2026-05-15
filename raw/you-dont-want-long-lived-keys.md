---
title: "You don't want long-lived keys"
url: https://argemma.com/blog/long-lived-keys/
date_fetched: 2026-05-14
section: "Security"
---

# You Don't Want Long-Lived Keys

## Key Arguments

**Why Long-Lived Keys Are Problematic:**
Three compounding risks: departing employees may retain key knowledge, attackers have increasing time to guess credentials, and cryptographic keys degrade in security effectiveness after extended use or high message volumes.

**The Ephemeral Keys Solution:**
Replacing long-lived credentials with short-lived alternatives (valid ~1 day or less) eliminates rotation pain since temporary credentials are inherently disposable. Converts a burdensome operational task into an automated feature.

## Practical Examples

- **SSH Access**: EC2 Instance Connect replaces hardcoded SSH keys with temporary credentials requiring current authentication
- **Package Publishing**: "Trusted publishers" allow GitHub Actions to generate temporary PyPI credentials instead of static tokens
- **Authentication**: SSO replaces user passwords with ephemeral signed assertions from identity providers

## Notable Quote
"Replacing long-lived keys with ephemeral keys is, for my money, one of the best uses of security engineering effort."

## Realistic Acknowledgment
Some long-lived keys remain necessary (like IdP signing keys), but minimize their count to concentrate security rigor where it matters most.

## Recommendations
1. Limit credential scope (e.g., encryption keys per customer shard)
2. Establish confident maximum key lifetimes
3. Rotate remaining keys quarterly minimum
4. Consolidate maintenance within specialized security infrastructure teams
