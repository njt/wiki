---
url: https://arxiv.org/html/2602.06547v4
title: '"Do Not Mention This to the User": Detecting and Understanding Malicious Agent Skills in the Wild'
author: "Yi Liu, Zhihao Chen, Yanjun Zhang (Griffith University), Gelei Deng (Nanyang Technological University), Yuekang Li (UNSW), Jianting Ning (Zhejiang Sci-Tech University), Leo Yu Zhang (Griffith University)"
date_fetched: 2026-07-21
date_published: 2026-06-10
---

A systematic security analysis of 98,380 skills from two community registries for LLM coding agents, using static pattern matching followed by sandboxed dynamic verification. The authors confirm 157 malicious skills containing 632 vulnerabilities across 13 attack techniques.

Two dominant, negatively correlated attack archetypes emerge. **Data Thieves** (70% of malicious skills) use remote code execution to exfiltrate credentials — one industrialized actor alone accounts for 54% of all malicious skills through templated brand impersonation with 100% template consistency. **Agent Hijackers** (10%) manipulate the LLM through adversarial natural-language instructions embedded in SKILL.md files, operating entirely at the instruction-following layer where traditional security tooling has no analogue.

The most striking finding: 84% of vulnerabilities are embedded in natural-language documentation (SKILL.md files), not in executable code. Each malicious skill averages 4 vulnerabilities spanning multiple kill-chain phases, and 73% contain shadow features — undocumented runtime behaviors invisible from public descriptions. Evasion scales with sophistication, though code-level obfuscation appears in only 9.5% of cases. Platform-native attack vectors include model substitution, supply-chain trojans, and hook-system weaponization.

Registry maintainers removed all 157 reported skills after disclosure, but the three-month undetected window demonstrates that reactive removal alone is insufficient. The authors release their dataset and detection pipeline publicly.
