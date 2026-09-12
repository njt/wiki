---
url: https://www.mendral.com/blog/you-should-not-update
title: "You should not update your dependencies in 2026"
author: Olivier Gambier
date_fetched: 2026-05-31
date_published: 2026-05-26
source_domain: mendral.com
reading_time: 11 min
topics:
  - security-and-sandboxing
---

# You should not update your dependencies in 2026

**Author:** Olivier Gambier (Mendral, ex-Docker distribution team lead)
**Published:** May 26, 2026
**Source:** https://www.mendral.com/blog/you-should-not-update

## Full Summary

Olivier Gambier argues that the traditional approach to dependency management — automatically updating everything as soon as patches are available — has become dangerous and counterproductive. The piece contends that dependency updates should be treated as untrusted code contributions requiring the same scrutiny as any pull request.

### The Death of the Old Security Model

Gambier traces his career back to the late 1990s, when the mantra was to "very carefully review what you depend on, read all changelogs and patches, apply timely, always be up to date." This approach worked in a smaller, more siloed tech world with few formal vendors. The massive shift to open-source — with its overworked, under-resourced maintainers — broke this model.

### Supply Chain Vulnerability → Supply Chain Compromise

The article traces an escalation: first came awareness of vulnerabilities in core dependencies (BIND, OpenSSL, Log4j), then came active compromise of the supply chain itself. Gambier argues that if everyone blindly trusts the supply chain, attackers will simply compromise maintainer accounts and ship weaponized code rather than hunting for existing vulnerabilities.

### Dependabot as Attack Vector

> "Blind installation and blind updating dependencies continuously (a-la dependabot)... became the number one vector of distribution for highly publicized supply chain compromises in the past 12 months."

The author calls for the death of "latest," soft tags, and unpinned version ranges.

### AI Agents Accelerating the Crisis

The rise of coding agents is described as having "turned the volume up to 11." More code is being produced and merged without meaningful review. Gambier notes that "the last standing, already frail, human safety guardrail, the good old code review, has been trampled and overrun."

### Humans Can No Longer Manage Supply Chain Security

> "Humans can no longer be in charge of the modern software supply chain security."

Gambier argues we are overwhelmed by non-actionable alerts and rarely review dependency patches.

### The Reactionary Path Is Dead

Efforts to revert to older, slower workflows (privately maintained forks, multi-month cooldowns, registry proxying) won't survive the industry's need for speed. "Going back is not a strategy."

### Mendral's Proposed Solution

When a Dependabot-style PR lands, an agent:
- Surfaces typo-squatting, known bad versions, OSV-flagged malware
- Assesses scrutiny level using openSSF scoring, publication patterns, ownership, and version age (versions <7 days old raise scrutiny; <72 hours max it out)
- Spins up a secure sandbox to inspect packages for compromise markers (new post-install scripts, new transitive deps, runtime behavior)
- Evaluates CVEs for reachability and blast radius in the specific repo context
- Audits posture weaknesses (unpinned actions, overprivileged tokens, soft tags, missing lockfiles)
- Comments on the PR with findings and can approve, request changes, or block

### Key Distinctions

- **Supply chain vulnerability** (finding bugs in dependencies) vs. **supply chain compromise** (actively weaponizing the distribution mechanism)
- **Old wisdom** (carefully vet and update) vs. **modern cargo-culting** ("just update everything, whatever it is, don't even look")
- **Reactionary approaches** (cooldowns, forks) vs. **programmatic verification** (AI-driven review on every dep update)

### Key Quotes

- "Damned if you do. Damned if you don't."
- "Open-source maintainers are not just free labor, they are also overworked, under-equipped, wildly understaffed"
- "Dependency updates must be considered as untrusted code contributions."
- "Going back is not a strategy."
- "Security should be our problem. And it is way too serious to be left to security professionals alone."
- "Trust nothing. Verify everything."
- "AI cannot outperform the best humans" — but excels at the mechanical, repetitive, volume-bound work of reading every diff
