---
url: https://dl.acm.org/doi/10.1145/3793655.3793739
title: "Execution-free Agentic Program Repair for Enterprise-Scale Development"
authors: "Saurabh Bodhe, Sanjukta De, Subhayan Roy, Jaydip Pokiya, Indira Vats, Sehajpreet Kaur, Xuemeng Li, Lejin Varghese, Yonas Bedasso, Max Kiehn"
date_fetched: 2026-07-25
date_published: 2026-04-12
venue: "FORGE '26 (IEEE/ACM Third International Conference on AI Foundation Models and Software Engineering)"
doi: "10.1145/3793655.3793739"
---

Automated program repair (APR) has advanced with large language models (LLMs), yet most systems rely on test execution for validation, which is impractical for industrial C++ codebases with complex dependencies and lengthy build cycles. We present PegasusAgent, an execution-free agentic APR system being deployed at AMD that autonomously performs ticket triage, code localization, patch synthesis, evaluation, and pull-request creation. The system combines ReAct-based reasoning with Model Context Protocol (MCP) clients, replacing test-based validation with static analysis (CppCheck) and semantic LLM critique during repair iteration, while ensuring production quality through final component builds and QA testing. Evaluated on real-world defects from AMD Radeon Software, the system demonstrates effective localization and patch generation. By leveraging organizational knowledge through similar-ticket retrieval and enabling rapid agent iteration without test execution overhead, this work advances APR toward practical enterprise-scale deployment in environments where traditional test-dependent approaches are prohibitively expensive. Comprehensive evaluations on the issues dataset show that our system outperforms baseline methods with correct file localization for 70.5% of issues and produce plausible fixes (score ≥ 0.50) for 33.1% of cases. Ablation studies further quantify the impact of individual system components, underscoring their respective contributions to overall performance gains.

CCS Concepts: Software and its engineering → Software defect analysis.

Keywords: agentic, APR, execution-free, industrial, LLM

## 1. Introduction

Modern software development workflows remain heavily dependent on manual labor across the issue-resolution lifecycle. When a defect is detected, engineers typically create an issue in tracking systems like Jira and proceed through repetitive steps of triage, code localization, patch development, build, and validation. Developers and QA personnel are often siloed into separate stages, with QA limited to reporting issues and validating fixes after integration. This separation elongates turnaround time and increases the likelihood of regressions slipping through.

Recent advances in LLMs have enabled significant progress in APR. However, most existing systems rely on test execution for validation, which is often impractical for industrial C++ codebases due to complex build dependencies, proprietary testing environments, and lengthy build cycles. Furthermore, current APR solutions are constrained by limited orchestration capabilities, minimal workflow customization, and weak integration of human feedback during intermediate repair iterations. They also lack mechanisms for leveraging organizational knowledge, such as historical bug-fix patterns and similar-ticket analysis.

The emergence of coding agents such as GitHub Copilot, OpenAI Codex, and Anthropic's Claude Code has established a foundational abstraction layer for LLM-assisted software engineering. While powerful as general-purpose assistants, they lack the specialized orchestration, domain-specific knowledge integration, and structured validation mechanisms required for fully autonomous defect resolution in enterprise environments.

To address these limitations, we introduce PegasusAgent for automated bug fixing at AMD — a concrete implementation built atop the coding agent abstraction that extends beyond interactive assistance to deliver end-to-end, auditable bug-fixing workflows.

### Key Contributions

- **Execution-free APR framework**: End-to-end issue resolution without relying on test execution, overcoming barriers in industrial C++ environments.
- **Context-aware localization and synthesis**: Similar-ticket retrieval and targeted code search enhances localization accuracy and patch relevance.
- **Hybrid patch evaluation**: Static analysis + LLM-based semantic critique for robust validation without executable tests.
- **Human-in-the-loop refinement**: Early QA involvement through local builds and iterative feedback loops shifts validation left.
- **Industrial deployment and evaluation**: Rigorously filtered dataset of 112 real-world QML/C++ defect repairs from AMD's development cycle.
- **Scalability and adaptability**: Designed to scale across different components and services within AMD.

## 2. Related Work

**Test-suite–based APR**: GenProg demonstrated genetic programming can evolve patches making failing tests pass. Repairnator showed how to operationalize APR as a CI bot that monitors failing builds, reproduces failures, attempts repair, and reports results.

**Agentic/LLM-based APR**: SWE-bench (2,294 real GitHub issues across 12 Python repos) assesses systems' ability to generate patches under execution-based evaluation. SWE-agent introduced an "agent–computer interface" (ACI) giving LMs repository-level actions. Recent evidence suggests SWE-Bench performance can be confounded by LLM memorization and benchmark contamination.

**Industrial evaluations**: Google's Passerine curated 178 internal bugs and adapted SWE-agent approaches to Google's environment. Good performance on machine-reported bugs but produced plausible patches for only a small percentage of human-reported bugs. Passerine's validation relies on bug reproduction tests, limiting applicability to industrial C++.

**Productized agents**: GitHub Copilot Coding Agent, OpenAI Codex, Anthropic's Claude Code — autonomous modes for taking assigned issues and returning tested PRs, but limited support for intermediate human-in-the-loop stages.

**Positioning**: AMD's system targets enterprise workflows spanning heterogeneous languages (QML/C++), integrates Similar Issue Context Retrieval to leverage organizational memory, and enforces structured orchestration via MCP-backed tools. Unlike benchmark-centric agents, the design emphasizes early QA participation through local build artifacts and iterative feedback, and combines rule-based gating (CppCheck) with LLM critique before PR creation.

## 3. Methodology: Agent Workflow

The proposed PegasusAgent implements an end-to-end agentic workflow combining ReAct-based reasoning with structured orchestration through MCP servers. Key modules:

### 3.1.1 Agent Instructions and Planning
The agent receives a base system prompt defining sequential workflow stages and enumerating available tools. Unlike traditional APR systems following rigid execution paths, the agent employs a planning-first approach: before taking any action, it autonomously generates a detailed execution plan.

**System Prompt (simplified):**
```
ROLE: Automated Program Repair Assistant

Workflow:
1. Analyze Issue
   1.a. Retrieve ticket description
   1.b. If screenshots supplied, extract relevant visual clues
   1.c. Query similar-ticket tool and infer common affected directory
2. Locate Relevant Code using repository tools
3. Identify Root Cause with clear technical explanation
4. Generate Fix as minimal, targeted changes (output as unified git diff)
5. Validate Solution via code evaluation tool; iterate until passes
6. Trigger Local Build only after evaluation success
7. Create Pull Request
```

### 3.1.2 Ticket Information Retrieval
Integrates with a Jira MCP server to automatically retrieve comprehensive ticket information including issue descriptions, reproduction steps, branch context, driver version, affected components, and historical QA feedback.

### 3.1.3 Code Localization (Two-Stage Pipeline)
1. **Similar Issue Context Retrieval**: Embeds ticket descriptions using OpenAI's text-embedding-3-large model and performs vector similarity search, filtering by program, triage category, and triage assignment. Returns up to 5 historically similar issues with associated code modifications.
2. **GitHub Code Search**: Using localized context from similar issues, constructs targeted GitHub code search queries through an agent-driven query builder.

### 3.1.4 Patch Synthesis
Combines ticket description, reproduction steps, and historical fix patterns with few-shot examples from similar-ticket analyses, enabling adaptation of repair strategies based on defect category (UI truncation errors, color issues, resource path corrections, etc.).

### 3.1.5 Patch Evaluation (Two-Stage, Execution-Free)
1. **Static Analysis Validation**: CppCheck performs token-level parsing, structural correctness checking, and compliance verification against C++ coding standards. Violations immediately disqualify the patch.
2. **LLM-as-a-Judge Validation**: A separate LLM evaluates whether the fix addresses the root cause, assesses potential side effects, checks coding conventions, and provides structured reasoning with a quality score. Low-scoring patches are rejected with feedback integrated into subsequent synthesis iterations.

### 3.1.6 Component Build and QA Feedback Loop
Upon passing validation, a new working branch is created from the target release branch. A remote build is triggered under production-equivalent configurations. QA reproduces the reported issue and verifies the fix. If issues are detected, structured feedback is ingested by the agent for subsequent repair iterations.

### 3.1.7 Pull Request Creation
The agent commits, generates a comprehensive PR description (ticket summary, localization rationale, applied changes, static analysis results, semantic critique, QA feedback history), and submits to the target repository with auto-assigned reviewers based on component ownership metadata.

## 3.2 Agent Tool Optimizations

A core contribution is an empirical study showing how pruning unnecessary tools and normalizing tool responses reduce context overhead and tool-use friction.

**Tool and response filtering**: Direct use of GitHub and Jira MCPs proved impractical — verbose responses with unnecessary metadata/HTML consumed context. A tool-filtering utility restricted the toolset to only six essential tools. Response filtering removed extraneous information and standardized outputs.

**PegasusAgent tool suite (MCP endpoints):**

| Tool | Inputs | Outputs |
|------|--------|---------|
| get_similar_tickets | Ticket ID; User input | Top-5 similar issues; File paths |
| code_evaluation | Ticket ID; code diff; base branch | accepted/rejected + rationale |
| trigger_build | Ticket ID; code diff | Path to executable |
| jira_get_issue_clean | Ticket ID | Ticket Description |
| search_code_filtered | Search query | Compact code matches |
| get_file_content_filtered | Repo name; owner; path | File paths/content + SHA |
| create_or_update_file | Repo name; path; content; message; branch | Commit SHA; file URL |
| create_branch | Branch name; repo name; owner | Branch ref |
| create_pull_request_clean | Repo name; owner; title; head; base | PR number; URL |

## 4. Experiments

**Model**: Claude Sonnet 4.5
**Retrieval**: Similar-ticket retrieval (top k=5)
**Filtering**: Tool filtering, Response filtering
**Output**: Patch explanation, diff, verification guidance for QA

### 4.1 Dataset

112 real bug fixes in QML and C++ from AMD Radeon Software production releases, selected through an 8-phase curation pipeline:

| Phase | Criterion |
|-------|-----------|
| 0 | Release window aggregation |
| 1 | Confirmed defect filtering |
| 2 | UI/UX component scope |
| 3 | Linked code submission required |
| 4 | Single-commit constraint |
| 5 | Single release branch scope |
| 6 | Single-file change constraint |
| 7 | QML / C++ language restriction |
| 8 | Heuristic revert–reapply curation |

### 4.2 Evaluation Metrics

1. **Code Localization Accuracy**: Exact file path match against developer's actual fix.
2. **Fix Quality (LLM-as-a-Judge)**: Similarity score 0-1 comparing agent patch against developer patch, covering structural similarity, functional equivalence, semantic coherence, code style, and conditional logic. pass@3: best patch among three attempts.
3. **Build and Testing Validation**: Whether the fix compiles and resolves the issue without regressions.

### 4.3 Baselines and Ablations

- **Full PegasusAgent**: Multi-step agent with similar-ticket retrieval, tool filtering, and response filtering.
- **Without Similar Issues Context**: Same pipeline but similar issues context retrieval disabled.
- **Without Tool Filtering**: Exposes the full GitHub/Jira MCP tool set.
- **Without Response Filtering**: Returns unprocessed server responses without normalization.
- **Claude Agent SDK**: Default Claude Code prompt with GitHub and Jira tools.
- **GitHub Copilot**: VSCode Copilot in Agent mode with GitHub and Jira tools.

## 5. Results

| Variant | Correct File | Wrong File | No Patch | Mean Score | ≥0.50 | ≥0.80 | # Tool Calls | Runtime |
|---------|-------------|------------|----------|------------|-------|-------|-------------|---------|
| Full PegasusAgent | 79 (70.5%) | 33 (29.5%) | 0 (0.0%) | 0.32 | 33.1% | 21.4% | 22 | 254s |
| w/o Similar Issues Context | 73 (65.2%) | 39 (34.8%) | 0 (0.0%) | 0.27 | 27.6% | 19.6% | 25 | 340s |
| w/o Tool Filtering | 60 (53.6%) | 39 (34.8%) | 13 (11.6%) | 0.25 | 23.2% | 15.2% | 19 | 399s |
| w/o Response Filtering | 26 (23.2%) | 34 (30.4%) | 52 (46.4%) | 0.13 | 9.8% | 2.7% | 22 | 398s |
| Claude Agent SDK | 43 (38.4%) | 54 (48.2%) | 15 (13.4%) | 0.14 | 13.3% | 8.9% | 14 | 295s |
| GitHub Copilot | 45 (40.2%) | 56 (50.0%) | 11 (9.8%) | 0.11 | 8.9% | 3.5% | 11 | 353s |

**Key findings:**
- Full PegasusAgent achieves strongest performance: 70.5% correct localization, 0.32 mean similarity, 33.1% ≥0.50 score.
- Disabling Similar Issue Context Retrieval drops localization from 70.5% to 65.2% and mean score from 0.32 to 0.27.
- Removing tool filtering degrades localization to 53.6% and increases no-patches to 11.6%.
- Eliminating response filtering causes a sharp performance collapse: 23.2% localization, 46.4% no-patch rate.
- Claude Agent SDK and GitHub Copilot baselines localize correctly in only 38.4% and 40.2% of cases.

### Qualitative Evaluation (5 Outcome Categories)

| Outcome | Count | Percent |
|---------|-------|---------|
| A (Identical to Developer Patch) | 7 | 6.25% |
| B (Human Interaction Needed) | 7 | 6.25% |
| C (Requires Local Build Verification) | 30 | 26.78% |
| D (Incorrect) | 58 | 58.03% |
| E (Image-Context Needed) | 3 | 2.67% |

### Online Deployment Metrics

Piloted with one team (11 developers + QA), tested over 2 weeks on production bug Jira tickets:
- 38.9% of cases: agent localized to correct files
- 27.8% of cases: correct fixes obtained
- 11.1% of cases: PR merged into codebase

## 5.1 Qualitative Examples

Five representative categories:
- **A**: Identical fix (e.g., icon source correction)
- **B**: Logic aligned but wording differs (e.g., tooltip text)
- **C**: Core intent correct but requires runtime verification (e.g., null guard vs. cached capability check)
- **D**: Incorrect, mislocalized, or edits unrelated file
- **E**: Image-driven visual alignment fix where agent adds superfluous visual adjustments

## 6. Conclusion & Next Steps

Future improvements: graph-based code search (capturing imports, invocations, inheritance), sub-agents for modular reasoning, finetuned coding agents and execution-free verifier models trained on internal codebases. Scope extension to backend logic, kernel-level programming, race condition detection, and performance optimization. Integration with CI/CD pipelines.

## Appendix A: Similar Issue Context Retrieval Analysis

Expanded evaluation on 1,657 tickets (removing phases 4, 5, 8 constraints):

| Level | Metric | Accuracy |
|-------|--------|----------|
| Component | Exact match | 96.02% |
| Top-level Directory | Exact match | 95.70% |
| Extension | Exact match | 71.92% |
| Directory Structure | Ratio of matching segments | 67.82% |
| Exact File | Recall@5 | 53.14% |

Key insight: Accuracy decreases monotonically from component-level (96%) to file-level (53%), reflecting increasing localization complexity at deeper directory nesting.

## Appendix B: LLM-Judge Evaluation

Uses OpenAI GPT-5 as the judge (temperature=1). Prompt defines five scoring dimensions: structural overlap, functional equivalence, scope alignment, minimality, and style/conventions. File mismatch, empty or non-code diffs, or purely extraneous changes mandate a score of 0. Human validation: two reviewers independently audited a stratified sample of 10 patches, confirming automated scores reflect meaningful alignment.

**Scoring interpretation**: ≥0.80 = near-identical/functionally equivalent; 0.60–0.79 = plausible with small omissions; 0.40–0.59 = partial overlap; <0.40 = divergent or noisy edits. pass@3 selects the highest score among 3 attempts.
