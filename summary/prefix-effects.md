---
title: "Prefix Effects"
url: https://antimemeticai.com/prefix-effects
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

Research demonstrating that semantic naming conventions significantly influence how AI agents generate code. Early naming decisions create a "gravity effect" that persists through subsequent development.

Vocabulary Crystallization: AI-generated codebases lock in naming patterns rapidly. TF-IDF similarity tracked across commits showed Gas Town shifted from 8% to 81% alignment in a single commit, while human codebases showed irregular fluctuations.

Prefix Propagation: Agents automatically extend prefixes to new functions within the same semantic domain. An agent seeing secure_create_user would generate secure_upload_document without instruction. Propagation weakened across domain boundaries.

Structural Divergence: Different prefixes produced measurably different code architectures. secure_ triggered password fields and bcrypt hashing despite no explicit mention. energetic_ produced 54% more decorators and asyncio usage. safe_ generated 53% more comprehensions and 11% more functions overall.

"A function name is a seed. secure_create_user makes an agent construct a world with passwords and bcrypt."

Naming conventions function as an "alignment surface" -- silently steering agent behavior through subsequent development while maintaining convergence on core infrastructure.