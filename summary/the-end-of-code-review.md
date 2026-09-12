---
url: https://arxiv.org/html/2606.13175v1
title: "The End of Code Review: Coding Agents Supersede Human Inspection"
author: Martin Monperrus
date_fetched: 2026-06-24
date_published: 2026-06-11
source: arXiv 2606.13175v1 [cs.SE]
topics:
  - agent-coding-workflow
---

# The End of Code Review: Coding Agents Supersede Human Inspection

**Author:** Martin Monperrus

**arXiv:** 2606.13175v1 [cs.SE], 11 Jun 2026

## Abstract

The paper argues that coding agents—LLM-based autonomous systems that read, write, test, and repair software—have crossed a capability threshold making traditional human code review unnecessary as a mandatory quality gate. The argument rests on two claims: (1) every goal of code review can be met by agents at lower cost and higher throughput, and (2) the naive model where agents write code and humans remain mandatory reviewers is a dead end that provides neither real assurance nor scales with AI-assisted throughput.

## I. Introduction

Code review serves four overlapping goals per Bacchelli and Bird (2013) and Sadowski et al. (2018): defect detection, style enforcement, knowledge transfer, and team awareness. However, it carries substantial costs—developers spend 10–15% of working hours on review, latency routinely exceeds 24 hours, and social friction occurs (tone escalations, seniority bias, first-time contributor abandonment).

The paper does **not** present new empirical work; it synthesizes existing capability evidence. Its three contributions are:

1. Demonstrating that all four review goals can be met by agents at lower cost/higher throughput
2. Arguing that AI code generation + mandatory human review is an unstable endpoint
3. A cost-benefit analysis showing the marginal defect-detection value of human review shrinks as agents improve

## II. Background

### II-A. Code Review: History and Practice
Originates with Fagan's 1976 paper on structured code inspections at IBM. Empirical evidence shows defect detection is "not what reviewers most reliably deliver"—style corrections, minor improvements, and questions dominate. Czerwonka et al. found reviews at Microsoft "do not find bugs" in the sense of deep logic-level defects; primary value lies in knowledge transfer and maintainability.

### II-B. Automated Code Review
Pre-LLM approaches used pattern matching and rule-based checkers. The LLM era brought systems like CodeReviewer, LLaMA-Reviewer, and CodeAgent. These systems focused on automating activities *within* a human-governed workflow. The author claims this paper is the first to argue for "complete displacement of mandatory human code review by coding agents."

### II-C. Coding Agents
Defined as an LLM embedded in an agentic loop—invoking tools for file I/O, shell commands, test execution, and iterative repair. Representatives include OpenAI Codex, Claude Code, SWE-agent, Devin, and GitHub Copilot.

## III. Evidence of Agent Capability

### III-A. Benchmark Performance
SWE-bench results show dramatic improvement: from ~1.7% (GPT-4 with retrieval) to ~12.5% (SWE-agent), exceeding 50% by late 2024, and over 70% by late 2025 on the public leaderboard. The author calls this trajectory "without precedent in the history of automated software engineering tools." AlphaCode ranked within the top 54% of humans on Codeforces contests.

### III-B. Review-Specific Capabilities
Pornprasit and Tantithamthavorn found agents detect the same defect categories human reviewers target. Li et al. showed CodeReviewer produces inline comments at quality comparable to trained human reviewers. Structural advantages of agents over humans include: holding full files, test suites, git history, and docs in context simultaneously; generating fixes and running tests in one loop; operating continuously regardless of time or day.

## IV. The Argument

### IV-A. Claim 1: Every review goal served by agents at lower cost and higher throughput

- **Defect detection:** Humans catch surface issues but are unreliable on deep bugs. Agents can perform "exhaustive dataflow reasoning" and cross-reference full test suites.
- **Style/standards:** Extends beyond linters to semantic style—naming conventions, idiomatic API usage.
- **Security:** Agents enumerate vulnerability classes more systematically than ad-hoc human review; AI scanners already outperform manual reviewers on standard benchmarks.
- **Knowledge transfer:** Agents generate on-demand explanations and updated documentation at merge time, more reliable than "incidental commentary of a busy colleague."

### IV-B. Claim 2: AI coding with human review is a dead end

Two failure modes:

1. **No genuine assurance:** When agents generate code, the independent-check assumption collapses. Subtle semantic errors are "invisible without running the full test suite." Reviews become rubber-stamps.
2. **Does not scale:** AI tools increase developer throughput, but human review capacity is fixed. The review queue becomes "the binding constraint on the delivery pipeline." The solution: an independent agent reviews agent-generated code, with humans intervening only for flagged uncertainty or explicit risk thresholds.

### IV-C. Claim 3: Cost-benefit calculation has flipped

The set of defects escaping agent review but caught by humans shrinks, while human review costs (time, latency, social friction) remain constant. The crossover point "has already been reached." Agent reviews are instantaneous, deterministic, auditable, and produce structured reports.

## V. Implications

### V-A. Implications for Software Engineering Practices

- **Merge workflow redesign:** Replace human gating with an agent-in-the-loop verification pipeline. Human approval reserved for high-risk changes, novel architecture, and regulated code paths.
- **Team structure:** Professional code reviewer role diminishes. Developers become "specifiers and orchestrators"—articulating requirements, evaluating outputs at higher abstraction. The craft shifts from "line-by-line inspection to system-level judgment."
- **Knowledge transfer:** Agents generate richer explanations than rushed human comments, but risk reducing informal bidirectional conversations. Complementary mechanisms (pair programming, mentorship) needed.
- **Open-source dynamics:** Agent review directly addresses maintainer bandwidth bottlenecks, compressing feedback loops and lowering barriers for first-time contributors.

### V-B. Implications for Tooling

- **CI/CD integration:** Review becomes a first-class engineering check analogous to compilation—runs automatically, produces structured reports (JSON, SARIF).
- **IDE/editor integration:** Agents review changes before commit, collapsing feedback from hours to seconds. Interaction shifts to natural-language conversation closer to pair programming.
- **Version control platforms:** Need cryptographically signed agent identities, audit log integration, structured UI components for agent-generated summaries and confidence scores rather than synthetic comment threads.

## VI. Discussion (Counter-arguments)

### VI-A. Hallucination and false negatives
Mitigations include ensemble review (multiple independent agents) and calibrated uncertainty reporting where agents abstain when unsure.

### VI-B. Security vulnerabilities in agent-generated code
Risk that generative and review blind spots are correlated. Mitigation: use "cyber-specialized frontier reviewers" for security sign-off, which can outperform traditional static analyzers.

### VI-C. Adversarial inputs and prompt injection
Maliciously crafted code could manipulate agent reasoning via embedded instructions. This is "a qualitatively new attack surface that does not exist for human reviewers" and requires treating prompt injection as a first-class threat.

### VI-D. Architectural coherence requires human judgment
The author counters that architectural coherence is best enforced through design documents and dedicated architecture reviews, not per-commit diff inspection, and empirical studies show reviewers focus on surface defects, not strategic architectural validity.

### VI-E. Ethical accountability requires human judgment
Agents are "not reliably equipped" to detect privacy violations or demographic bias. The author counters that mandatory PR review is "not the appropriate locus for ethical scrutiny"—that belongs in requirements engineering and post-deployment monitoring. Human escalation reserved for changes agents flag as uncertain or high-risk.

## VII. Conclusion

The author argues code review's role will "refocus to a layer of high-stakes human oversight"—architecture decisions with long-lived consequences, security-critical paths in regulated systems, and changes depending on requirements agents haven't been given. For everything else, mandatory human review is "difficult to defend on technical grounds." The paper concludes: "The end of code review, as we once thought the absolute best practice of modern software engineering, is the beginning of a more productive way to build software."

## Key References

32 sources spanning Fagan (1976) on formal inspections, Bacchelli & Bird (2013) on modern code review goals, Sadowski et al. (2018) on Google's practices, Czerwonka et al. (2015) on reviews not finding bugs, SWE-bench benchmarks, CodeReviewer, and various security and productivity studies.
