---
url: https://arxiv.org/html/2605.20049v1
title: Does Code Cleanliness Affect Coding Agents? A Controlled Minimal-Pair Study
author: Priyansh Trivedi, Olivier Schmitt (SonarSource)
date_fetched: 2026-07-08
date_published: 2026-05
---

# Does Code Cleanliness Affect Coding Agents? A Controlled Minimal-Pair Study

## Authors
**Priyansh Trivedi** and **Olivier Schmitt** — SonarSource

## Abstract

The paper investigates whether the structural and stylistic quality ("cleanliness") of code affects an agent's ability to navigate and modify it. The authors introduce an evaluation protocol based on minimal pairs of repositories that share architecture, dependencies, and external behavior but differ on static-analysis rule violations and cognitive complexity. Across 660 trials with Claude Code (using Claude Sonnet 4.6), cleanliness did not change pass rates but did alter the agent's operational footprint: "7 to 8% fewer tokens" and a "34%" reduction in file revisitations. The central claim is that "traditional maintainability principles remain highly relevant in the era of AI-driven development."

## 1 Introduction

The authors note that autonomous coding agents are seeing rapid adoption. A 2026 survey of 128,018 GitHub projects found traces of agent activity in 22–29% of them. Running agents is expensive — a single SWE-bench Verified task averages roughly 4 million tokens.

The authors identify a gap in existing work: evaluations focus on pass rates or token consumption, but always hold the codebase fixed while varying the agent or scaffolding. No prior work had held the agent and task fixed while varying the codebase itself.

Three research questions are posed:
- **RQ1:** Does code cleanliness impact task completion success?
- **RQ2:** Does it change the operational footprint (tokens, reasoning, conversation length, files read, revisitations, lines edited)?
- **RQ3:** Does the effect vary between single-dense-region tasks and multi-module tasks?

The authors state their three contributions: (1) a protocol for controlled measurement using bidirectional minimal-pair construction, (2) a benchmark of six minimal-pair repositories with 33 tasks and hidden tests, and (3) findings that while pass rate doesn't change, cleaner code reduces token usage by 7–8% and revisitations by roughly 34%.

## 2 Benchmark Construction

The benchmark consists of minimal pairs of repositories that are behaviorally equivalent but differ on cleanliness. Cleanliness is quantified via SonarQube rule violations. The benchmark uses the Harbor framework and defines behavioral equivalence as passing "the same test suite at equal coverage."

### 2.1 Constructing Minimal Pairs

Two pipelines are used:

**Degradation (Slopify):** Takes a clean codebase and produces a degraded version — code that might have grown without code review or linting. It runs in three phases: Build (get tests passing), Explore (write summaries of directories), and Transform (introduce SonarQube rule violations while keeping tests passing).

**Cleanup (Vibeclean):** Takes an organically messy codebase and resolves issues. The agent works through the analyzer's flagged issues module by module, running tests after each to ensure behavioral parity.

Six minimal pairs were assembled — three Java and three Python. Three are public, three are private (to guard against LLM memorization). The transformations include inlining helpers, duplicating logic, padding with dead code (degradation), and deduplicating strings, deleting dead code, removing dead branches, and extracting helpers from god structures (cleanup).

**Table 1** shows the six repositories with metrics: sonar-sca (private), sonar-caas-poc (private), sonarcloud-codedatalake (private), commons-bcel (public), genie (public), and ckan (public). Issue densities range from 0.73 to 49.46 per kLOC across sides.

### 2.2 Designing Tasks

Three rules guide task design: (1) route through hotspots where the two sides differ most, (2) describe tasks in externally observable terms (no file or function names), and (3) test at the public surface.

Tasks were generated through a five-phase process: comparing variants to map differences, creating task outlines, manual curation, converting outlines into actionable tasks with hidden tests and reference implementations, and automated checks requiring reference implementations to pass tests on both sides and untouched repositories to fail them.

The final benchmark has 33 tasks across three tracks:
- **13 Cognitive-hotspot tasks** — routed through dense single-method/class complexity
- **14 Multi-module tasks** — requiring changes across two or more modules
- **6 Calibration tasks** — cleanliness-insensitive, adding a log line or similar

An example task from genie asks the agent to "add a structured failure-stage tag to Genie's synchronous job-launch timer."

## 3 Experimental Setup

All experiments use Claude Code with its default toolset, running Claude Sonnet 4.6. The agent is blind to which side of a pair it faces. Each of the 33 tasks is run 10 times per side, producing 660 trials total.

Each trial runs in a containerized sandbox with bounded CPU, memory, storage, and wall-clock budgets. Within each pair, sandboxes differ only in the source tree.

Ten metrics are recorded: Pass rate, Input tokens, Output tokens, Reasoning characters, Conversation turns, Turns before first edit, Characters before first edit, Files read, File revisitations, Lines edited.

An outlier filter drops trials more than 50% off the median within each (task, side) combination, removing 9.7% of trials. Dataset-level numbers are micro-averaged.

## 4 Results

### 4.1 Aggregate Findings

Cleanliness changes footprint without changing pass rate. The dataset-level pass rate difference is -0.9 percentage points (0.913 cleaner, 0.921 messier) — essentially flat.

Footprint metrics shrink on the cleaner side: input tokens fall 7.1%, output tokens fall 8.5%, reasoning characters drop 11.1%. The authors note this suggests "the agent is doing the same work with fewer tokens."

Trajectory metrics shift in the same direction but less strongly: conversation messages drop 7.0%, messages before first edit drop 3.6%, characters before first edit drop 4.6%.

The largest single effect is file revisitation: "on the cleaner side, the agent returns to files it has already edited 34% less often." This holds across every repo. Files read increases slightly (+3.2%), which the authors interpret as the agent reading "wider on its first pass" on cleaner code versus returning to verify edits on messier code.

### 4.2 Per-Track Token Footprint

- **Calibration track** (6 tasks): Pass rate is flat. Token and files-read deltas sit within a few percent of zero.
- **Multi-module track** (14 tasks): The strongest contrast — input tokens drop 10.7% and revisitations drop 50.8%. Distinct files read is essentially unchanged.
- **Cognitive-hotspot track** (13 tasks): Token footprint is essentially unchanged. However, on the cleaner variant the agent opens more files (+11.2%) and edits fewer lines per file. This is consistent with cleanup pipelines redistributing "complexity across more methods rather than eliminating it."

### 4.3 Variance and Per-Task Spread

Trial-to-trial variance is substantial. "The most expensive trial in a group typically costs around 2.5× the cheapest," with roughly 72% of groups having a ratio above 2×. The most extreme case is ckan/organization-list-exclude-empty, where cleaner-side trials span 1.4M to 10.6M input tokens.

Task-to-task spread is also wide. Per-task input-token deltas span -47% to +44% with a median of -4.5%. On 16 of 27 real tasks, cleaner code uses fewer input tokens; on 11, it uses more.

### 4.4 Case Studies

**Case 1 — Cleaner wins (commons-bcel/stack-effect-annotation-in-disassembly):** The messier variant has opcode dispatch in two parallel god methods with several-hundred-line switches. The cleanup pipeline replaced these with thin dispatchers delegating to named helpers. On the cleaner repo, agents use 35% fewer input tokens, open 25% fewer files, edit 31% fewer lines, and take 32% fewer turns. Because the agent can grep precisely where edits need to happen, it navigates "in a more token-efficient manner."

**Case 2 — Messier wins (genie/cluster-active-job-limit):** The cleanup pipeline extracted helpers around focal launch logic but kept the original logic in place. The file is roughly the same size on both variants, but the cleaner side spreads surrounding work across more methods. This shows up as an 8% increase in input tokens on the cleaner variant.

The contrast: on BCEL, the cleanup replaced god dispatchers and agents could easily grep; on Genie, the pipeline "left the focal logic intact and added structure around it, and the agent paid for the added surface area."

### 4.5 Comment-Volume Ablation

The authors address a confound: the minimal-pair pipeline also moves comments and suppression markers. In sonar-caas-poc, the cleaner version has 8,000 more comment lines than the messier version; the slopify process removed docstrings and explanatory blocks.

After normalization:
- sonar-caas-poc moves from +1.2% to -18.0% on input tokens
- sonar-sca moves from +1.3% to -8.5%
- sonarcloud-codedatalake and genie show negligible change

The authors conclude comments and suppression markers are not driving the contrast. However, they note normalization was involved and variable, so they cannot make a stronger claim.

## 5 Related Work

Discussed: SWE-bench and SWE-bench Verified as establishing the task format; SWE-Effi introducing resource-aware metrics; Tokenomics dissecting ChatDev on GPT-5 (finding 59.4% of tokens consumed in code review); AgentTaxo taxonomizing multi-agent systems with 2:1 to 3:1 input-to-output ratios.

Most closely related: Bai et al. (2026) measuring token consumption on SWE-bench Verified, finding "agentic tasks consume roughly 1000× the tokens of single-turn reasoning." The authors corroborate two findings (input dominance, per-task variance).

Two papers vary code while holding model/task fixed: Le et al. (2025) on identifier names and summarization accuracy; Simoes and Venson (2025) on code readability attributes. Both "build their minimal pairs at the file or snippet level" and probe single-turn LLM calls, whereas this work uses multi-turn coding agents over full repositories.

SlopCodeBench (Orlanski et al., 2026) is described as a long-horizon counterpart where agent-generated code "drifts generally toward more verbose, less structured output, ending 2.3× longer than the human-maintained baselines."

## 6 Limitations

Seven limitations identified: (1) author-curated end to end, (2) one configuration (only Claude Sonnet 4.6), (3) tokens not dollars, (4) hidden tests only, (5) no post-facto cleanliness scan, (6) labels are within-pair relative, (7) short-horizon — unknown whether effects compound across a year of agent work.

## 7 Conclusion

The paper pushes back against the claim that maintainability principles lose relevance as agents take over development. Across six minimal pairs and 33 tasks, cleaner code did not change task completion but reduced the agent's footprint. The clearest behavioral signal was file revisitation decreasing by roughly a third. The authors conclude that "the principles that make code easier to maintain for a human developer appear to do the same for an agent," while leaving open whether per-task improvements compound long-term.

## References

1. Bai et al. (2026) — Token consumption in agentic coding tasks (arXiv:2604.22750)
2. Chowdhury et al. (2024) — Introducing SWE-bench Verified (OpenAI Blog)
3. Du et al. (2024) — Evaluating LLMs in class-level code generation (ICSE)
4. Fan et al. (2025) — SWE-Effi (arXiv:2509.09853)
5. Harbor Framework Team (2026) — Harbor framework (GitHub)
6. Jain et al. (2025) — LiveCodeBench (ICLR)
7. Jimenez et al. (2024) — SWE-bench (ICLR)
8. Le et al. (2025) — When names disappear (arXiv:2510.03178)
9. Oracle & OpenJDK Community (2025) — OpenJDK
10. Orlanski et al. (2026) — SlopCodeBench (arXiv:2603.24755)
11. Qian et al. (2024) — ChatDev (ACL)
12. Robbes et al. (2026) — Agentic much? Adoption of coding agents on GitHub (arXiv:2601.18341)
13. Salim et al. (2026) — Tokenomics (MSR)
14. Simoes & Venson (2025) — Code readability attributes and LLMs (SBES, arXiv:2507.05289)
15. Wang et al. (2025) — AgentTaxo (ICLR Workshop)
