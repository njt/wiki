# Execution-Free Agentic Program Repair

AMD's PegasusAgent is the most honest industrial APR paper yet published: an execution-free automated bug-fixing agent that achieves 70.5% file localization and 33.1% plausible fixes on 112 real QML/C++ defects from the Radeon Software production cycle, while openly reporting that only 6.25% of patches match the developer fix exactly and 58% are incorrect. The real contribution isn't the APR pipeline — it's the empirical proof that organizational memory (similar-ticket retrieval), tool pruning, and response normalization matter more than model capability for industrial agent performance. Remove response filtering and the system collapses from 70.5% localization to 23.2% with a 46.4% no-patch rate.

---

## Key Quotes

> "Most existing systems rely on test execution for validation, which is often impractical for industrial C++ codebases due to complex build dependencies, proprietary testing environments, and lengthy build cycles."

This is the paper's unspoken thesis: the academic APR literature optimized for a world that doesn't exist in industry. SWE-bench tests whether you can fix Python repos with runnable test suites. AMD's developers work in QML/C++ with no tests, cross-language references, and builds that take longer than a coffee break. The execution-free constraint isn't a research novelty — it's table stakes for deploying APR anywhere that matters.

> "Removing tool filtering—thereby exposing the full GitHub and Jira MCP toolsets—further degrades localization accuracy to 53.6% and increases runtime to 399s per issue. This confirms that uncontrolled tool exposure inflates context length and burdens reasoning."

The most damning table in the paper (Table 3) is the ablation study. Full PegasusAgent: 70.5% localization. Remove response filtering: 23.2%. That's a 47-point swing from infrastructure choices, not model quality. The paper doesn't quite say it, but the implication is clear: **most agent performance papers are measuring prompt engineering and tool curation, not model capability.** The gap between Claude Agent SDK (38.4%) and full PegasusAgent (70.5%) is 32 points, and none of it comes from a better model — it's all workflow design, tool filtering, and organizational memory.

> "Evaluations on the issues dataset show that our system outperforms baseline methods with correct file localization for 70.5% of issues and produce plausible fixes (score ≥ 0.50) for 33.1% of cases."

Read this carefully: 70.5% of the time the agent finds the right file. But only 33.1% of the time does it produce a fix scoring ≥0.50 against the developer's fix. And only 6.25% of fixes are identical. The gap between "finds the right file" and "writes the right fix" is where the real work is — and the paper is refreshingly transparent about it. Most APR papers would bury these numbers; AMD leads with them.

> "We observed that our issue descriptions contain significantly fewer code-related terms and identifiers, requiring more localization effort. Approximately 70% of our internal dataset issues contain fewer than two code terms, while 41.8% of issues in SWE-Bench contain more than 5 code words."

A data point that should reshape how we talk about SWE-bench validity: AMD's real-world bugs have almost no code terms in their descriptions. SWE-bench issues are practically annotated with line numbers by comparison. If SWE-bench performance is confounded by memorization (as [12] argues), this distribution gap is the mechanism — real bugs don't come with file paths in the description.

> "It was observed that in 38.9% of cases the agent was able to localize to correct files. Correct fixes were obtained for 27.8% of the tested tickets. In 11.1% of cases the PR was merged into the codebase."

The deployment numbers are lower than the benchmark numbers — localization drops from 70.5% (curated dataset) to 38.9% (live production tickets). This is the lab-to-production gap in one sentence pair, and the paper reports it without flinching.

---

## Key Themes

- **#pattern** Execution-free validation: Replace test suites with CppCheck (deterministic static analysis) + LLM-as-a-Judge (semantic critique). The hybrid approach gives you a fast deterministic gate plus a slower semantic gate — the pattern generalizes beyond APR to any domain where tests are expensive or absent.

- **#concept** Organizational memory as localization engine: Similar-ticket retrieval (OpenAI embeddings + Weaviate vector DB + categorical filters) narrows the search space from an entire monorepo to a handful of candidate files. Component-level accuracy is 96%; file-level is 53%. The vector DB of historical fixes is the moat.

- **#pattern** Tool and response pruning as first-class engineering: The MCP protocol gives agents dynamic tool discovery, but the paper shows that *too many tools* and *too-verbose responses* are actively harmful. The custom middleware layer (tool filtering → 6 essential tools, response filtering → strip HTML and metadata) is not optimization — it's the difference between a working system and a 46.4% no-patch rate.

- **#tool** MCP server architecture: Three custom MCP servers (CodeGen, GitHub-proxy, Jira-proxy) plus middleware for tool/response filtering. The insight: official MCP implementations aren't production-ready for agents — they expose too many tools and return too much noise. The fix is proxy servers, not better models.

- **#person** Human-in-the-loop refinement with QA shift-left: Instead of QA validating at the end, QA participates iteratively during repair. Structured feedback ("Fix resolves issue but introduces visual artifact in panel X") feeds back into the agent loop. This collapses the traditional developer-QA silo.

- **#concept** Honest benchmarking: The 5-category outcome taxonomy (A-E, from identical to incorrect) and the gap between curated-dataset results (70.5% localization) and live-deployment results (38.9% localization) is a model for how industrial ML papers should report results. The paper tells you what works, what partially works, and what doesn't — in production, not just in the lab.

---

## Critical Analysis

**The real contribution is the infrastructure, not the repair.** The paper frames itself as an APR contribution, but the most transferable findings are about agent infrastructure: tool pruning as a 32-point localization swing, response normalization as a 47-point swing, organizational memory as a 5-point swing. If you're building an industrial coding agent, these three levers — not model choice — are where you should spend your engineering time. The paper essentially proves that Claude Sonnet 4.5 with the right scaffolding beats Claude Agent SDK or GitHub Copilot with the same model but generic scaffolding by a factor of 2× on localization and 2.5× on fix quality.

**The execution-free validation pattern has legs.** CppCheck → LLM judge → QA review is a generalizable pipeline for any domain where tests are sparse or expensive. The paper shows it works for UI defects (QML/C++); the same pattern should apply to infrastructure-as-code, database migrations, configuration changes, and documentation — anywhere correctness is semantic rather than behavioral. This is the paper's most exportable idea, and it's buried in Section 3.1.5.

**The dataset curation is the unsung contribution.** The 8-phase funnel from release window → 112 clean single-file fixes is a research contribution in itself. Every industrial APR paper should steal this pipeline. The heuristic revert-reapply integrity check (Phase 8) — where they verify that reverting the fix cleanly reconstructs the buggy state — is the kind of defensive engineering that separates a usable benchmark from a misleading one.

**The honesty about failure is the paper's superpower.** 58.03% of patches are incorrect (Category D). Only 6.25% are identical to the developer fix. The live deployment merge rate is 11.1%. These numbers would kill a startup pitch deck, but they're the most useful data in the paper. They tell you where APR actually stands in 2026 for industrial C++: useful for localization (70% chance of finding the right file), occasionally useful for fix generation (33% chance of a plausible patch), rarely fully autonomous (6% chance of a perfect match). This is the honest baseline that every subsequent industrial APR paper should compare against.

**What's missing:** The paper doesn't discuss cost economics — what does it cost per ticket to run the full pipeline vs. a human developer? At 254s per issue with Claude Sonnet 4.5, the token costs are non-trivial for a team processing hundreds of tickets. There's also no discussion of failure modes beyond "incorrect" — what happens when the agent produces a subtly wrong fix that passes CppCheck and the LLM judge but introduces a regression only visible at runtime? The QA feedback loop catches some of these, but the paper doesn't quantify the false-negative rate of the static+semantic validation pipeline.

**The MCP middleware story deserves more attention.** The paper's custom GitHub and Jira MCP servers with tool-filtering middleware and response normalization is effectively a pattern language for making MCP production-ready. Most organizations adopting MCP will hit exactly these problems — too many tools, too-verbose responses, HTML instead of plain text — and the paper provides a battle-tested solution. It's worth reading alongside Anthropic's official MCP guidance and the [[Building Agents for Production Systems with MCP]] page.

---

*Sources: [[raw/pegasus-agent-execution-free-apr]]*
*Last updated: 2026-07-25*
