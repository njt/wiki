---
url: https://dsifry.github.io/harnesseval/
date_fetched: 2026-10-08
---

## Find more bugs with the model you already use

Your AI code reviewer may not need a smarter model. It may need a better workflow. The same models found 1.6× as many verified bugs when a harness ran the review.
That was the median gain from adding a harness across 42 comparisons in an AI code review study. A harness runs the model through several review passes, with tools and specialist reviewers, instead of asking it to review the code in a single prompt.
→ More verified bugs in 39 of 42 comparisons with the same model and effort level
→ Best harness setup: 88 bugs found. Best single-prompt setup: 54
→ GLM-5.3 with metareview at low effort found 2.1× as many bugs as its single-prompt setup
The gain has a price: median token use was about 10× higher, cost 5× higher, and review time 3.5× longer. There were also more unsupported findings to check. A higher bug count does not automatically mean less work for a developer.
Use a harness. Then measure how much time its findings save on your own code.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#f-harness
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
What would change if your reviewer found more bugs with the model you already use?

## A comparable score for a fraction of the cost

Why pay $2.97 for an AI code review when $0.22 matched its quality score? GLM-5.3 matched Opus 5 using the same harness, at 1/13 the cost.
GLM-5.3 running metareview at low effort matched Opus 5 running the same harness and effort in this study. The score rewards verified bugs and useful advice, and penalises unsupported findings.
→ GLM-5.3: $0.22 per review, 72 bugs found across the test set
→ Opus 5: $2.97 per review, 74 bugs found
→ Both rounded to the same quality score: 0.567
The cheaper model is open-weight: its weights are available for others to host, subject to its license. The result is a reason to choose an open-weight setup for value, not a claim that all models perform equally.
Speed matters too. The open-weight setup took 95 seconds per review at low effort, but 38 minutes at medium and 49 minutes at high. Keep effort low for a fast review. Check the host’s data policies before sending code.
These are list-price estimates. Close scores do not establish equal performance on other workloads; run your own evals.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#f-cost
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
What would your team do with that difference in review cost?

## Make high effort earn its bigger bill

Do you set your AI to High, Max or Ultra and wonder why the results aren’t satisfactory?
In 17 of 22 AI code review comparisons, the study could not establish a quality difference between high and medium effort. High cost more in 20 of 22.
→ High effort significantly improved the quality score in four comparisons
→ It significantly reduced the score in one
→ Costs ranged from 0.96× to 4.2× the medium-effort price
The score here is F1: it balances bugs found against false alarms. This effort comparison used the original 42 benchmark labels, rather than the expanded verified-bug set. An uncertain result does not mean the settings are identical. It means this sample cannot tell them apart.
Default to low or medium effort. Make high effort earn its extra cost on your own code. High versus low was not formally tested, and each setup ran only once, so this is not proof that extra reasoning never helps.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#f-effort
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
Has your team measured what high effort actually buys?

## See what changes when the same model gets a harness

The model gets all the attention. The review workflow deserves more. Harnesses beat single prompts in 39 of 42 comparisons—with the same model and effort.
This chart compares a single review prompt with a harness: several review passes, tools, and specialist reviewers. Each row keeps the model the same. The marks average its results across effort levels.
→ The harness found more verified bugs in 39 of 42 same-model, same-effort comparisons
→ The average gain was 13.5 percentage points in recall—the share of verified bugs found
→ The best harness setup found 88 of 147 bugs; the best single-prompt setup found 54
It was not a win in every comparison. The three losses were all Sonnet 5 running Compound Engineering. Harnesses also used more tokens and produced more unsupported findings to check.
The single prompt was carefully written, with a reviewer role, bug categories, severity levels, and instructions to report only confident findings. Its exact text is in the technical report.
The practical takeaway: the way a model reviews code deserves as much attention as the model you choose.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#c-harness
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
Have you compared a harness with your current review prompt?

## Find the review setup that fits your budget

AI code review for $0.024 a pull request sounds too cheap to be useful. That setup found 71 of 147 verified bugs across the test set. Here’s what paying more actually bought.
That was GLM-5.3-Flash running metareview at low effort in this AI code review study. The bug count is a total across the selected pull requests; the price is the average per review.
This chart puts quality against price:
→ Highest score: Opus 5 · Compound Engineering · medium, 88 bugs, $6.12 per review
→ Lowest cost to find at least 70 bugs: GLM-5.3-Flash · metareview · low, $0.024 per review
→ Fewer unsupported findings than that budget pick: GLM-5.3 · metareview · low, $0.22 per review (5 against 19)
Left is cheaper. Up is a higher score. The line connects the best score available at each price. All but its most expensive point are open-weight models, whose weights are available for others to host under their licenses.
The score rewards bugs and useful advice, and penalises unsupported findings. The leading scores have overlapping uncertainty ranges. Use the chart to choose a setup, then measure its results and the time spent checking findings on your own code.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#c-money
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
Which matters most for your team: more bugs found, fewer false alarms, or a lower bill?

## See where high effort helped—and where it did not

“Think harder” is easy advice when someone else pays the bill. High effort cost more in 20 of 22 comparisons, but significantly improved review quality in just four.
This AI code review chart compares high and medium effort for the same model and review method. It shows the change in F1, a score that balances bugs found against false alarms, using the original 42 benchmark labels.
→ Green: a statistically significant improvement
→ Red: a statistically significant decline
→ Grey: the sample cannot establish a difference
The bar shows uncertainty across pull requests. If it includes zero, high effort may have helped, done nothing, or hurt. Grey does not mean “no effect,” and it does not tell you the cost.
Cost is shown separately on the right. High effort cost more in 20 of 22 comparisons, with ratios ranging from 0.96× to 4.2× across all comparisons. Each setup ran once; the bars do not include variation from repeating the same run.
Default to low or medium effort. Use your own evaluation to decide when a higher bill buys a result worth paying for.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#c-effort
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
What evidence would make you turn high effort on?

## Check your own model choices against the data

Is your AI code reviewer worth what you pay? Compare 66 setups side by side—bugs caught, unsupported claims, price, and speed. See where your setup stands.
This interactive explorer lets you compare models, review methods, and effort levels on the same pull requests. A harness runs several review passes with tools and specialist reviewers; the alternative is a single review prompt.
→ Filter to the models you care about
→ Compare bugs found, quality scores, useful advice, and unsupported findings
→ Put those results against dollars, seconds, or tokens per review
→ Pin setups side by side and share the comparison
The study’s main findings: harnesses found more bugs in 39 of 42 comparisons; GLM-5.3 running metareview matched Opus 5 at 1/13 the cost; high effort often cost more without a clear improvement over medium.
Every number comes from the same data as the technical report. Bug counts are totals across the test set; costs and times are averages per review. Small score differences are not reliable rankings, and the study did not measure developer time saved.
Choose a setup from the evidence, then run your own evals on your own workloads.
The source: Dave Sifry’s open harnesseval study of AI code review. It compared eight models, a single review prompt, and the free harnesses Compound Engineering and metareview at three effort levels. The main comparison used six selected pull requests from two codebases, with 147 verified bugs and a single run per setup. Disclosure: Sifry wrote metareview. The data and code are public.
Finding and chart: https://dsifry.github.io/harnesseval/#c-explore
Technical report, methods and data: https://dsifry.github.io/harnesseval/REPORT.html
Where does your current reviewer land?
