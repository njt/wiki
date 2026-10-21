---
url: https://www.laude.org/updates/lab-notes/revengine
date_fetched: 2026-10-21
---

## Contents6

At Laude, we’ve been reworking our internal software development process to reflect that code is now cheap. One of the biggest pain points in this era of generated code has been reviewing pull requests: coding agents now churn out more code than teams have human attention to evaluate.

To that end, we’ve been working on a tool to streamline our review process so we can spend human attention where it’s most valuable. We call this tool RevEngine (GitHub). It is open-source, Apache-licensed and runs as a GitHub Action.

## Why RevEngine?

The seeds of RevEngine were planted the week after we publicly announced our open-source agent harness Headlong. After the announcement, we saw an influx of pull requests from outside contributors. It quickly became apparent that a full manual review of each pull request to ensure:

- it did what it claimed,
- it advanced the goals of Headlong, and
- it didn’t introduce vulnerabilities or other security issues

would not scale with our small maintainer team. Not only would we miss things as we tried to keep pace with the incoming diffs, but time spent reviewing pull requests would directly trade off with time spent working on our research agenda.

We considered off-the-shelf code review tools; it’s an active space with many entrants. However, we concluded that a minimal harness and a strong model would get us quite far (a throughline in much of our work) and give us much more flexibility when it came to a repo’s particular needs.

Thus the earliest analogues of RevEngine were born. We first tested skills for Claude Code and Codex, then a repo with its own tooling to orchestrate reviews. Finally, we consolidated our learnings into the RevEngine GitHub Action so we could install it on our other public and internal repos.

## How it works

The figure below illustrates how RevEngine works. RevEngine gives a strong model read-only access to the repository and asks it to review a pull request. The model proposes findings after it reads the diff, the surrounding code, and the project’s rules and design documents. A second pass rechecks each finding against the code context and drops findings it judges as incorrect or unsupported. After deduplicating findings, those remaining are ranked by severity and confidence. With Anthropic’s Opus 5.5, a review of a typical pull request takes about 3 minutes and costs about $1 in tokens.

## Evaluating performance

We ran our tool against GitHub’s recent ReviewBench benchmark. ReviewBench aims to measure how well AI agents find issues in real-world pull requests, as compared to human-validated ground truth. It contains about 2,600 human-reviewed problems from 219 pull requests in 187 different repos.

When the results of our official full benchmark run went public on October 16, 2026, we were pleasantly surprised to see that RevEngine topped the ReviewBench leaderboard. Our run raised the top F1 score from 48% to 60%.

ReviewBench leaderboard

Grounded scores · review-bench.ai · October 19, 2026

| # | Agent | F1 | F1 bar | Precision | Recall | Duration | 
|---|---|---|---|---|---|---|
| 1 | Laude RevEngine anthropic/claude-opus-5.5 Oct 14, 2026 | 59.6% | 83.6% ± 0.5 | 46.3% ± 1.0 | 3m 2s ± 4s | |
| 2 | block/review sol-opus · off Oct 14, 2026 | 47.7% | 88.7% ± 0.6 | 32.6% ± 0.5 | 3m 17s ± 2s | |
| 3 | Klaussy claude-sonnet-5-5 · 1 Oct 13, 2026 | 44.1% | 84.1% ± 0.4 | 29.8% ± 0.5 | 3m 5s ± 8s | |
| 4 | Copilot code review Balanced Oct 1, 2026 | 40.1% | 87.8% ± 0.5 | 26.0% ± 0.6 | 4m 43s ± 18s | |
| 5 | DeepSeek Harness Reviewer deepseek-flash · high · 2400 Oct 14, 2026 | 39.9% | 89.5% ± 0.1 | 25.6% ± 0.9 | 5m 19s ± 39s | |
| 6 | Devin AI Sep 28, 2026 | 37.0% | 84.0% ± 0.7 | 23.8% ± 0.1 | 3m 13s ± 9s | |
| 7 | Qodo Sep 28, 2026 | 35.1% | 85.3% ± 0.8 | 22.1% ± 0.9 | 4m 34s ± 46s | |
| 8 | Codex GPT-5.6 Sol · Max Oct 8, 2026 | 30.4% | 86.0% ± 1.1 | 18.5% ± 0.4 | 9m 28s ± 19s | |
| 9 | Cubic Jun 29, 2026 | 27.3% | 85.5% | 16.3% | 11m 42s | |
| 10 | Greptile Jun 16, 2026 | 27.2% | 86.1% | 16.2% | 4m 49s | |
| 11 | Cursor Sep 27, 2026 | 17.3% | 87.7% ± 0.8 | 9.6% ± 0.3 | 4m 16s ± 18s | 

**Figure 2.**RevEngine on the ReviewBench public leaderboard.

RevEngine was also the fastest agent by a few seconds, which makes it the only point on the Pareto frontier of F1 against average review time.

## Where does the performance come from?

We wanted to understand our results on ReviewBench. Was our score just a consequence of using a strong model, or did our harness also contribute?

To find out, we ran naive baselines with four models (Anthropic’s Opus 5.5, Sonnet 5.5 and Haiku 5.5, and DeepSeek V4 Flash) on the 25 pull requests in ReviewBench’s maintainer-selected test set. This naive baseline was one model call that could see only the pull request’s title, description, and diff, with the instruction to report the problems it finds. It could not open other files or read the project’s rules, and nothing checked its findings. We then compared these results to the RevEngine harness with each of those models on the same 25 pull requests.

Our experiments suggest that a strong model sets baseline performance, while the harness makes reviews more thorough. Adding RevEngine raised F1 for every model we tried, by as much as 14 points for Sonnet. It enabled Opus to catch about a third more of the real issues with nearly the same precision. It also increased the share of high-severity problems Opus caught from 32% to 55%.

The RevEngine harness also helped the stronger models more. Haiku gained 8 points and DeepSeek V4 Flash gained 5, against 11 to 14 for Opus and Sonnet. Our read is that a stronger model can better take advantage of the harness’s scaffolding to improve its review.

Surprisingly, switching from Sonnet to Opus added only one point to F1 under RevEngine; Sonnet nearly matched Opus at half the cost. Opus did keep a slight lead on high-severity problems, where it found 55% against Sonnet’s 48%.

## Future work

While our ReviewBench performance is encouraging, the real test is how RevEngine slots into our development process and whether it reduces time spent manually reviewing pull requests without impacting the stability of our projects. We are collecting this data now.

## What did we learn?

Here’s what we learned building RevEngine:

- **Write the project’s rules down.**We prompt RevEngine to read design docs and rules so it adapts its behavior to each repo. As more software engineering processes are handed over to agents, writing down charters, rules, and design principles in the codebase becomes more valuable. These documents let an automated reviewer ask whether a change is correct- *and*whether it belongs in the codebase.
- **Match reviewer pickiness to context.**RevEngine lets you choose which severities get reported. People get worn down by long lists of nits and multiple rounds of back and forth, but coding agents don’t care whether they fix two things or ten. A reviewer can be stricter when an agent opens a PR (but watch out for unbounded nit spirals). This same tension shows up in measuring performance on benchmarks like ReviewBench: tools that hide low severity findings to spare human contributors may score lower if the benchmark expects thorough reports.
- **Review is cheap compared with attention.**About $1 and 3 minutes per pull request with Opus is orders of magnitude cheaper and faster than a software engineer. Leaning on automation can keep engineers from burning out under a flood of pull requests, instead focusing on higher-value work.
- **Keep a person on the dangerous parts.**You likely still want a human to check the diff before merging changes to the riskiest parts of your codebase. Where to draw the line depends on the project.

We’re a small team with more projects than people, so we’re excited to keep developing RevEngine as part of a broader rethink of how we build software. Give it a try in your own repo today and let us know how it works.
