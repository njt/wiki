# Pensieve (Ted Haley)

Pensieve is a free, MIT-licensed macOS tool that indexes your files, git repos, worktrees, and coding-agent sessions on-device and serves them to agents over MCP as targeted passages instead of grep-then-read-everything. It is also, quietly, an argument: that the biggest lever on agent cost and quality isn't a smarter model but a better retrieval layer, and that agents grep by habit rather than by choice.

---

The product pitch is "one search points them to the few passages that matter." The benchmark is the interesting part: headless Claude Code, 72 held-out tasks across four real repos (504 to 34,847 files), run twice. 131/144 correct file finds vs 123/144 for grep, 2.0 vs 3.4 tool calls per lookup, $0.041 vs $0.050 per lookup, and — the honest bit — plain grep still wins on exact error messages.

## Key quotes

> Agents are only as good as the context they're given.

The framing the whole page rests on. It positions Pensieve not as a search tool but as a context-supply tool — the retrieval half of the [[Agent Memory and Context]] problem, priced in tool calls and dollars.

> Agents grep by habit. The install prompt runs `pensieve integrate recommended`, which adds a short "search before you grep" section to the instructions files of Claude Code, Codex and Gemini CLI, plus a search skill and two hooks for Claude Code.

This is the quietly important detail. A retrieval tool is worthless if the agent doesn't call it, so the product ships steering files and hooks alongside the MCP server. Tool adoption here is a prompt-engineering problem as much as an engineering one — convergent with the lesson in [[Steering Claude Code]] that instruction-delivery mechanism matters as much as the instruction.

> The tasks were never used to tune Pensieve. Costs are Claude Code's reported API costs at list prices... because the cost depends on Claude's prompt cache at the time.

Unusually candid benchmarking for a product page: open scripts, held-out tasks, a stated confound (prompt-cache timing), and a concession where a rival (exact-match grep) wins. The numbers are small-N and vendor-run, but the methodology is respectable and reproducible.

> When the people who wrote a part of the codebase are no longer active in it, that knowledge is at risk.

The bus-factor feature — git-blame ownership maps, knowledge-gap flagging, "pair with Priya before changing tax rules" — reaches past agents back into ordinary engineering practice. It's the same territory as codebase archaeology, except computed continuously instead of during an audit.

## Themes

#tool #concept #pattern

- **Retrieval as the cheapest context win.** A semantic index over code, docs and past sessions beats brute-force grep-and-read on tool calls, tokens, and latency — most of the time.
- **Tool adoption is a steering problem.** The MCP server alone isn't the product; the `integrate` step that edits instruction files and installs hooks is what makes agents actually use it.
- **Local-first as a design stance.** On-device embeddings, no account, secrets never indexed, optional AI that can run on Qwen locally or LM Studio. Privacy as default rather than a policy page.
- **Blame as knowledge graph.** git blame weighted by surviving lines becomes a live map of who knows what — organizational intelligence from repository metadata.

## Take

This is a vendor page, so the 18%-cheaper headline deserves skepticism — the benchmark is self-run, four repos, 144 runs, with a confessed prompt-cache confound. But it's the most honest self-benchmark of its kind I've seen on a product page: open scripts, held-out tasks, and a visible loss to grep on exact error strings. The more interesting bet isn't the percentage, it's the architecture: sessions as first-class searchable sources, worktrees indexed per-branch, uncommitted edits searchable seconds later. That is a bet that an agent's *local* context — your files, your past agent conversations — is the highest-value retrieval corpus, which complicates the cloud-memory approach of tools like [[Claude-Mem]] and the session-retrieval patterns in [[Recoll]].

The "search before you grep" instruction block is also a small real-world experiment in a bigger question: can you steer a model off its training-data habits with three sentences of instructions? Their benchmark says the agent used Pensieve in all 144 runs once steered — which is either a triumph of steering or a reminder that what works on a vendor's own tasks is the easiest thing to demonstrate.

## Related pages

- Strengthens [[Building a RAG Pipeline for Semantic Code Search (JetBrains)]] — both argue semantic code retrieval beats grep-driven exploration; Pensieve adds a shipped, benchmarked product form and the past-sessions-as-corpus idea.
- Complicates [[Maybe Coding Agents Don't Need a Bigger Memory]] — Pensieve's bet is that agents don't need bigger memory either, just better-filtered access to what's already on disk; the 43% tool-call reduction is evidence for filtering over accumulating.
- Nuances [[Scaling LLMs to Larger Codebases]] — the savings grow with repo size (biggest repo, biggest saving), a concrete mechanism for the codebase-scale context problem.
- Shares the local-first instinct of [[Recoll]] — desktop full-text search — but adds agent-facing MCP delivery and embeddings, showing where classic desktop search ends and agent context begins.

---
*Sources: [[raw/pensieve]], [[summary/pensieve]]*
*Last updated: 2026-10-12*
