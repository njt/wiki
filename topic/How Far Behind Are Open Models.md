# How Far Behind Are Open Models

Håvard Tveit Ihle quantifies the open/closed model gap with real data: **8–10 months on private benchmarks, 4–6 months on public ones**. The gap was narrowest around DeepSeek R1 (Jan 2025) and has been widening since. The analysis is careful about contamination risks and provider degradation bias — the headline number is likely an *underestimate* because closed labs have more varied data, more enterprise customers, and less incentive to overfit to benchmarks. The real gap on actual tasks is probably wider.

> "On private benchmarks, open models trail the closed frontier by roughly **8–10 months**. On public benchmarks, the gap is roughly **4–6 months**. The gap was smallest around the time of DeepSeek R1, in Jan 2025, and since then the gap has been growing."

The public/private gap asymmetry is the most important finding. It tells you the 4–6 month figure you'd get from glancing at public leaderboards is wrong by roughly a factor of two. The real measure — benchmarks where answers can't be memorized — shows a wider and growing gulf.

> "Third-party providers can have subtly degraded performance when serving open models."

This is a quiet bombshell for anyone running evals on open models through API providers instead of raw weights. Your gap measurements might be inflated not because open models are worse, but because your inference pipeline is subtly broken. The author flags this but can't quantify it.

> "Well-resourced closed labs probably have more access to varied data, more enterprise customers, and less need to focus on benchmark scores."

This is the structural explanation for why the gap widens. Open model developers optimize what's measurable. Closed labs optimize what customers pay for. Over time, those vectors diverge — and decoupling from public benchmarks is itself a moat.

> "The risk is over- not under-statement of the gap."

After auditing all 17 benchmarks for trustworthiness, Ihle concludes the methodological biases all point one way. Self-reported scores, FrontierMath's OpenAI-funded provenance, and ARC-AGI's semi-private contamination all inflate the closed side. The analysis is already conservative.

## Themes

- #concept **Public vs. private benchmark gap** — The gap nearly doubles when you look at non-public data. Public benchmarks overstate how close open models are.
- #pattern **Benchmark overfitting as structural disadvantage** — Open model developers optimize benchmark scores because that's their signal. Closed labs optimize enterprise utility. These vectors diverge.
- #concept **Backward-looking gap methodology** — Associating the gap with the open model's release date (not the closed model's) avoids bias from currently-open gaps. Clever statistical hygiene.
- #tool **Epoch AI Benchmarking Hub** — Primary data source. The analysis was mostly coded by Claude Opus 4.7.
- #pattern **Provider degradation bias** — Third-party inference providers may silently degrade open model performance, especially Chinese models. A measurement problem masquerading as a capability problem.

## Critical Analysis

This is one of the most careful benchmark-gap analyses I've read. The methodology is transparent about its own weaknesses — the provider degradation caveat, the contamination audit, the backward-looking framework. It doesn't oversell.

What's under-discussed: the gap isn't just about model weights. It's about the *ecosystem* — RLHF infrastructure, evaluation pipelines, and training data access. Open-weight releases give you the artifact, not the factory. The widening gap since DeepSeek R1 suggests the factory matters more than we thought.

The piece is also refreshingly honest about what it *can't* measure. Real-world task performance is likely worse than any benchmark suggests, and the author says so. No spurious precision.

The key strategic question the data raises: if the gap keeps widening, does open-weight AI become a permanent second tier? Or is this just a lag that closes whenever a well-funded open effort (like DeepSeek) ships? The data can't answer that, but it frames the question sharply.

## See Also

- [[2025 in LLMs]] — Simon Willison's annual survey of the LLM landscape
- [[Recent Developments in LLM Architectures]] — KV sharing, attention budgeting, and other cost-reduction techniques that might close the gap from the bottom
- [[Granite 4.1]] — IBM's open-source family with documented RL training
- [[Step 3.7 Flash]] — 97% of Opus 4.6 at 1/9th cost; per-harness benchmarking across six scaffolds
- [[Muse Spark]] — Meta's first proprietary frontier reasoning model
- [[Notes from the AI Now Summit by Mistral]] — "Model alone isn't enough" thesis
- [[Demystifying Evals for AI Agents]] — The eval problem from the agent builder's perspective
- [[stupidmeter]] — AI benchmarking tool with deliberately retro UI
- [[Benchmark Exploitation]] — When benchmarks become targets they stop being useful measures

*Source: [LessWrong](https://www.lesswrong.com/posts/rJcCrXyEsJKmmDpWG/how-far-behind-are-open-models), Håvard Tveit Ihle, 2026-05-28. Fetched 2026-06-05.*
