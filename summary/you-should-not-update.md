---
url: https://www.mendral.com/blog/you-should-not-update
title: "You should not update your dependencies in 2026"
author: Olivier Gambier
date_fetched: 2026-05-31
date_published: 2026-05-26
topics:
  - misc
---

# You should not update your dependencies in 2026

**Author:** Olivier Gambier (Mendral, ex-Docker distribution team lead)
**Published:** May 26, 2026
**Reading time:** 11 min

## Summary

Gambier traces the arc from 1990s sysadmin culture — where manually vetting and patching a handful of known vendors was feasible — to the present day, where massive open-source ecosystems, overworked maintainers, blind automated updates, and now AI-generated code have created a crisis of trust in the software supply chain. His core thesis: **dependency updates should be treated as untrusted code contributions**, not auto-merged. Current tools like Dependabot, once helpful, have become primary attack vectors. Humans are overwhelmed and can't scale. Reactionary measures (private forks, cooldown periods) won't survive industry velocity. The solution, he argues, is AI-assisted review that lives *inside* the CI pipeline — doing the mechanical, pattern-heavy work of examining every dependency diff, evaluating reachability of CVEs, and flagging behavioral drift.

## Section 1: The simpler times…

Gambier opens with a personal anecdote: his first server running phpMyAdmin, late 1990s, compromised within two weeks because he didn't know patches existed. The sysadmin model of that era — manually vet a handful of known vendors (BIND, OpenSSL, Apache) — actually worked at that scale. You could read the release notes. You could test before deploying. The trust model was personal and auditable.

## Section 2: From supply chain vulnerability to supply chain compromise

The transition: from known vulnerabilities in known software (patchable) to compromised packages distributed through trusted channels (unpatchable by conventional means). The key insight is that the attack surface shifted from *the software* to *the supply chain itself*.

Key incidents cited:
- Log4j (Log4Shell) — the watershed moment
- Shai Hulud, Nx s1ngularity, axios, TeamPCP, chalk/debug/qix, tj-actions/changed-files — all within the past 12 months

The argument is that we've crossed a line: we can no longer distinguish between "upstream published a new version" and "an attacker published a new version through upstream's compromised credentials."

## Section 3: Judgment day

The current state is described as "Damned if you do. Damned if you don't." Not updating leaves you vulnerable to known CVEs. Updating exposes you to supply chain compromise. There is no safe default.

Key contributing factors:
- Open-source maintainers are "overworked, under-equipped, wildly understaffed"
- Dependabot and similar tools "were genuine and major progress five to ten years ago. Now they are just harmful"
- AI coding agents have introduced unprecedented volumes of code into the supply chain, compounding the problem
- "The institution of the last standing, already frail, human safety guardrail, the good old code review, has been trampled"

## Section 4: What we should be saying out loud

### Humans can no longer be in charge of modern software supply chain security
The scale is beyond human capacity. The average project has hundreds or thousands of transitive dependencies. No team can manually review every update to every dependency.

### Rigid automated-updates tooling cannot stay in charge either
Dependabot blindly merges version bumps. It can't evaluate whether a change is malicious. It was designed for a trust model that no longer holds.

### Dependency updates are untrusted contributions
This is Gambier's core reframe. Every dependency update should be treated the same way you'd treat a PR from an unknown external contributor — with skepticism and review. The automation that currently auto-merges these changes is the vulnerability.

### The reactionary path is a dead end
Private forks, manual review of everything, refusing to update — these are "going back" strategies that can't scale to industry velocity. "Going back is not a strategy."

### "Securing" AI tooling is not the next AppSec frontier
Gambier argues against the framing that "securing AI" is the new security frontier. The frontier is the *same* one — supply chain security — but AI has made it exponentially harder and also provides the only viable solution.

## Section 5: What Mendral is building

Gambier describes their product: an AI agent that lives inside CI and:

1. Reviews every dependency change on every PR — flagging typo-squatting, known bad versions, OSV-flagged malware
2. Assesses scrutiny level based on openSSF scoring, upstream publication patterns, ownership, open issues, and **age of the version** (< 7 days raises scrutiny; < 72 hours maxes it out)
3. Above scrutiny threshold: spins up a secure sandbox, downloads and inspects the package for compromise markers
4. Comments on PRs with findings — approves, requests changes, or blocks
5. For known CVEs: evaluates reachability and blast radius in the specific repository context
6. On every branch: audits for posture weaknesses (unpinned actions, overly privileged tokens, soft tags, missing lockfiles)

On the question of trusting AI for security: "AI is not a magic wand and (at least for now) cannot outperform the best humans." But it can do the mechanical, pattern-heavy review work at a scale humans can't, freeing human reviewers to focus on what they're actually good at.

## Section 6: So, should you actually not update?

The conclusion: no, you should update — but you should treat every update as an untrusted contribution and review it accordingly. The slogan "you should not update" is a provocation, not literal advice. The point is that the current default of auto-merging dependency bumps is no longer safe.

## Key Quotes

- "The old operating model was indeed fine in a much smaller, simpler tech world"
- "Literally the first thing we deeply internalized in that era was to 'very carefully review what you depend on'"
- "Damned if you do. Damned if you don't."
- "open-source maintainers are not just free labor, they are also overworked, under-equipped, wildly understaffed"
- "Dependabot and siblings were a genuine and major progress five to ten years ago. Now they are just harmful."
- "Dependency updates must be considered as untrusted code contributions."
- "Going back is not a strategy."
- "The institution of the last standing, already frail, human safety guardrail, the good old code review, has been trampled"
- "Trust nothing. Verify everything."
- "…we just gave up anyhow and basically wait for an upstream patch while the stuff is embargoed"
- "AI is not a magic wand and (at least for now) cannot outperform the best humans."

## References Cited

- Ken Thompson, "Reflections on Trusting Trust" (1984)
- Eelco Dolstra's PhD thesis on package managers
- Russ Cox's research on dependencies
- Simon Willison's "Zig anti-AI" post (April 2026)
- Wiz blog on CVE-2026-3854
- NVD operations update (April 2026)
- openSSF, socket.dev
- Andrea Luzzardi's post on agent harness

## Author Context

Olivier Gambier is co-founder of Mendral (AI DevOps Engineer platform). Previously led the Docker distribution team in 2014, rebuilding Docker's image distribution on content-addressability. Co-founders include Sam Alba and Andrea Luzzardi (both ex-Docker/Dagger).
