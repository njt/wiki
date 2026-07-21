---
url: https://arxiv.org/html/2602.06547v4
title: '"Do Not Mention This to the User": Detecting and Understanding Malicious Agent Skills in the Wild'
author: "Yi Liu, Zhihao Chen, Yanjun Zhang (Griffith University), Gelei Deng (Nanyang Technological University), Yuekang Li (UNSW), Jianting Ning (Zhejiang Sci-Tech University), Leo Yu Zhang (Griffith University)"
date_fetched: 2026-07-21
date_published: 2026-06-10
---

# "Do Not Mention This to the User": Detecting and Understanding Malicious Agent Skills in the Wild

## Abstract

The paper presents a systematic security analysis of 98,380 skills from two major registries for LLM-based coding agents. Using static pattern matching and dynamic behavioral verification, the authors identified 157 skills with confirmed malicious behavior, covering 632 vulnerabilities across 13 attack techniques. They found each malicious skill averaged 4.03 vulnerabilities spanning multiple attack phases. Two dominant, negatively correlated attack strategies emerged: credential theft via remote code execution and agent manipulation through adversarial instructions. Over half of cases came from "a single threat actor employing templated brand impersonation at scale." Following disclosure, "registry maintainers removed all 157 (100%) of the reported skills."

## 1. Introduction

LLM-based coding agents increasingly rely on third-party extensions called **skills**—file-based packages that bundle natural language instructions (SKILL.md) with helper scripts running at full user privilege. The authors note that "installing a skill typically grants it full local user privileges, with minimal scrutiny or interactive confirmation." Attackers embed reverse shells or malicious instructions within skill files, a vector already exploited in real-world campaigns like GTG-1002.

The study constructs the first large-scale labeled dataset of malicious agent skills. The pipeline narrows 98,380 skills → 4,287 suspicious candidates → 157 behaviorally-confirmed malicious skills with 632 labeled vulnerabilities.

Three research questions drive the work:
- **RQ1:** Threat landscape characterization
- **RQ2:** Attack strategies and coordination
- **RQ3:** Detection evasion methods

Key contributions include a labeled benchmark dataset, multi-dimensional threat characterization, and ecosystem impact validation (100% removal after disclosure).

## 2. Background and Related Work

The paper distinguishes agent skills from other LLM extension mechanisms (in-context instructions, function calling, MCP). Unlike MCP, "skills execute locally with user privileges, exposing both code-level and instruction-level attack vectors." Community registries indexed over 98,000 skills within three months, "typically without security review."

The authors compare against browser extension security, IDE extension risks, and package manager supply-chain threats (npm/PyPI). Agent skills differ because they "run with pre-granted user privileges, thereby eliminating the exploitation stage" and introduce instruction-level attacks through natural language files.

Prior work (Liu et al., Cisco's Skill Scanner) flagged suspicious patterns but could not distinguish malicious intent from benign functionality — the gap this study addresses through behavioral verification.

## 3. Methodology

### 3.1 Definitions and Threat Model

**Operational definition:** A skill is malicious when it exhibits behaviors like credential theft, data exfiltration, remote code execution, privilege abuse, agent manipulation, or hidden functionality — with evidence of intentional abuse.

**Threat model:** The adversary is a skill publisher. Three attacker goals are data theft, agent hijacking, and persistence. Two attack vectors exist: code-level (bundled scripts) and instruction-level (SKILL.md directives).

### 3.2 Data Collection

Skills were collected from two community registries in January 2026:
- **skills.rest:** 25,187 skills from 2,337 GitHub repos
- **skillsmp.com:** 73,193 skills from 8,909 repos
- **Total:** 98,380 skills

### 3.3 Malicious Skill Candidate Identification

The authors mapped attack behaviors to an adapted MITRE ATT&CK framework with 6 kill chain phases (Reconnaissance, Credential Access, Execution, Defense Evasion, Exfiltration, Impact) and defined 14 detection patterns. Code-level patterns used regex matching; instruction-level patterns used GPT-5.2-based analysis. Static analysis flagged 4,287 candidates (4.4%).

### 3.4 Behavioral Verification

All 4,287 candidates ran in sandboxed Docker containers with network monitoring, system call tracing, file auditing, and honeypot credentials. Of these, 762 triggered runtime indicators. Two authors independently reviewed all 762; inter-rater agreement reached 94.5% (Cohen's κ=0.89). Confirmed malicious: 157 skills (3.7% of candidates; 0.16% of total corpus).

### 3.5 Vulnerability Labeling

Each confirmed skill's vulnerabilities were extracted and assigned severity levels (CRITICAL/HIGH/MEDIUM/LOW) following OWASP's agentic framework. This produced 632 labeled instances across 13 patterns (of 14 defined; E4—Network Reconnaissance—never appeared). Inter-rater agreement was κ=0.91.

**Shadow features** (undocumented runtime behaviors not inferable from public descriptions) were also identified: 115 of 157 skills (73.2%) contained them.

### 3.6 Co-occurrence Attack Chain Identification

The authors constructed co-occurrence matrices, used Fisher's exact tests with Bonferroni correction, applied Louvain community detection, and stratified skills into three sophistication levels (Basic: 1-2 patterns/no evasion; Intermediate: 3-4 patterns/evasion or shadow features; Advanced: 5+ patterns with both evasion and shadow features).

## 4. Measurement Study

### 4.1 RQ1: Threat Landscape

**Attack Scope:** "Malicious skills average 4.03 vulnerabilities each, with 71.8% rated CRITICAL or HIGH severity." Only 19.7% had 1-2 vulnerabilities; 80.3% bundled three or more. The three most prevalent patterns were SC2 (Remote Script Execution, 25.2%), P4 (Behavior Manipulation, 18.8%), and E2 (Credential Harvesting, 17.7%).

**Natural language attack surface:** "84.2% of vulnerabilities in our dataset (532 of 632) are embedded in SKILL.md files" — the natural language documentation. Executable code accounted for only 8.5%.

**Kill chain coverage:** The median skill spans 3 of 6 phases. Execution (75.2%), Impact (69.4%), and Credential Access (68.2%) dominate.

**Sophistication stratification:** The ecosystem is "middle-heavy" — 77.7% at Level 2 (Intermediate), 15.9% at Level 1, 6.4% at Level 3. Vulnerability density increases 2.6× from Level 1 to Level 3.

### 4.2 RQ2: Attack Strategies

**Data Exfiltration Chain (E2→E1):** Credential harvesting and external transmission co-occur in 58 of 157 skills (36.9%, OR=2.24, p=0.020).

**Negative association reveals two archetypes:** SC2 and P1 show significant anti-correlation (OR=0.11, p<0.001, φ=-0.41):

- **Data Thieves** (SC2-centered): 110 skills (70.1%) with SC2 without P1. Enriched for E2 (OR=23.8) and E1 (OR=9.7). Supply chain exfiltration strategy.

- **Agent Hijackers** (P1-centered): 16 skills (10.2%) with P1 without SC2. Their intent is control, not data theft.

**A single industrialized actor** (smp_170) accounts for 54.1% of all malicious skills (85 skills) through templated brand impersonation with 100% template consistency (26 identical lines across every skill). The E2+SC2 fingerprint identifies this factory with OR=556. The 85 impersonated brands span 15 industry sectors. This single publisher contributed 58.5% of all 632 vulnerabilities.

### 4.3 RQ3: Detection Evasion

**Shadow features** increase monotonically with sophistication: 0% (Level 1) → 86.1% (Level 2) → 100% (Level 3). Most evasion operates at the documentation level, not code level.

**Code-level obfuscation** appears in only 15 skills (9.5%) — Base64, marshal, hex encoding.

**Platform-native attack vectors** (n=6 skills) target the AI platform's own infrastructure:
- Model substitution (man-in-the-middle at the model level, redirecting API calls)
- Supply chain trojan (98.3% text similarity to a legitimate skill with 3 injected lines)
- Hook system weaponization (PreToolUse/PostToolUse monitoring and exfiltration)
- Sleeper cells triggered by codewords

## 5. Discussion

### 5.1 Two Archetypes, Two Threat Models

Data Thieves have "reached commodity malware maturity" — the smp_170 factory is identifiable with OR=556. Agent Hijackers present a qualitatively different challenge: "These 16 skills operate entirely within the LLM's instruction-following layer." Traditional security tooling "has no analogue for a Markdown file that manipulates an AI to act against its user."

### 5.2 Lessons from Mature Ecosystems

The authors compare against browser extensions (circa 2012 structural parallel but lower barrier — "writing a phishing email" vs. writing an exploit), Android malware (comparable kill chain coverage achieved in 3 months vs. years, due to collapsed privilege model), and npm/PyPI supply chain (technique parallel but novel attack surface in natural language).

### 5.3 Defense Implications

Key recommendations include natural-language-first detection, parallel detection pipelines for the two archetypes, and platform-native hardening (permission scoping, MCP credential rotation, hook sandboxing). The 100% removal rate "validates responsiveness but not prevention" — the three-month undetected window demonstrates reactive removal's insufficiency.

### 5.4 Limitations

The dataset is a January 2026 snapshot. The 60-second dynamic analysis window may miss time-delayed payloads. Sophisticated attackers may detect the sandbox. The 157 confirmed skills "represent a lower bound" — a 10% sample of unconfirmed candidates found 93.2% contained dormant triggers. smp_170's dominance means aggregate statistics should be interpreted with the per-publisher Gini coefficient of 0.71 in mind.

## 6. Conclusion

The study presents "the first large-scale measurement of the malicious agent skill ecosystem" with 157 behaviorally-confirmed malicious skills and 632 vulnerabilities. The ecosystem has bifurcated into Data Thieves and Agent Hijackers. One industrialized actor accounts for 54.1% of activity. Evasion scales with sophistication. All reported skills were removed. The dataset and pipeline are publicly released to support future research.
