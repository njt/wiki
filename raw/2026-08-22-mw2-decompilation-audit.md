---
url: https://web.archive.org/web/20260925153131/https://momo5502.com/posts/2026-08-22-mw2-decompilation-audit/
date_fetched: 2026-10-11
---

In the last post I presented my multi agent decompilation setup for MW2, almost framed as a scream for help. I received great suggestions that I will gradually implement over the next few days.

However, to get insights on what actually happened and how well the previous setup performs, I started evaluating the session logs (~2GB).

## Where the cost goes

This project uses a subscription, so costs are actually at about 200€. However, we would be at $85,207 on pay-per-token API pricing.

| Worker | Overseer | |
|---|---|---|
| List-price-equivalent spend | $70,275 | $14,933 | 
| Tool calls | 198,143 | 38,173 | 
| Assistant turns | 364,786 | 81,705 | 

We’re at about 89% cache reads, which is not super surprising, given that agents run 24/7, non-stop.

## Where the time goes (Worker agents only)

Out of 1107.6 total worker agent-hours, this is how time was spent (in %):


### Idle time

The idle time is especially annoying, because it’s pure time wasted.

In the beginning, I used `/goal` to keep the agents working over a longer timespan. However they had a tendency to prematurely mark the goal as completed.

I have since switched to `/loop` which creates a cron job that fires every 3 minutes to remind the agents of their work.

Combined with other smaller measures (e.g. creating a memory that the harness takes care of context and they never have to idle, thinking their context is full) this seems to have almost entirely eliminated idle times:

### Tool calls

The individual tool calls (command executions) also reveal room for optimization:

#### Build

Time spent building is the highest number, although it is worth noting that 35% of all build calls chained a test run. So a big part of the time spent building is actually spent running the tests.

Unfortunately, optimizing time spent building is hard. Compiling the generated code is important to catch mistakes early. Changes in central headers can cause full recompilations. However, as the code structure is defined by the game, there is not much to change here. Agents used to compile Release builds, as that had warnings-as-errors enabled and I wanted the agents to catch those errors. This has since been switched to Debug, with a CMake option to enable warnings-as-errors on Debug builds. This yields a 30% reduction of the time spent building.

#### Shell search/read

Time spent searching and reading is probably not really optimizable.

#### CI polling

Time spent polling CI is a complete waste. It makes sense that agents react to CI results, as CI runs tests on other platforms (Linux) and runs all configurations. However, actively polling CI is completely useless. I have since set up a webhook on GitHub that notifies the agents of CI failures (as mentioned in the previous post). I hope this completely eliminates the time spent there.

#### Sleep/wait

Time spent sleeping is almost as bad as idle time. The agents mainly sleep awaiting GitHub events. After a push, they wait for CI to start (when they used to poll it), or to check if an issue they were working on was closed. By adding `Fix #N` in the commit message, GitHub auto-closes the issue. Agents polling for that is useless. Adding a reminder to their PostCompact hook to never recheck issues for closing might help. We’ll see if it really does.

#### Tests

Time spent testing can be useful, but it’s also something I’m trying to reduce. Most test runs are successful, meaning the time is wasted. Given that CI failure notifications are now dispatched to the agents, test executions are almost redundant. As a matter to prevent that, test execution has been disabled in the code. Agents should only execute them when reproducing CI failures, or when adding new tests. We’ll see if that helps.

#### git

Git operations are necessary and not worth optimizing.

#### Other

Interestingly, Powershell has a 17% error rate, way above any other tool (most under 2%). Discouraging the agents from using Powershell, or even outright disabling its execution, might be a logical consequence.

## Context

| Metric | Value | 
|---|---|
| Total compactions | 269 | 
| Context size before compacting (median) | 933,088 tokens | 
| Assistant turns between compactions (median) | 1,346 | 
| Time between compactions (median) | 3.3h | 

Sessions live up to 93-100% of the 1M context window before compacting.

As sessions spend much of their time near the limit of the context window, almost every turn pays close-to-1M-token cache-read cost.

Most of the data in the context is volatile in this project: An already decompiled function is largely irrelevant for further turns.

That means earlier compaction could save a lot of tokens, yet, result in nearly equal performance.

I have tuned the harness to perform compaction at 60% of the context window now (as opposed to 93%). We’ll see if this saves tokens.

## Subagents

Initially, workers were using lots of subagents.

They can not perform actual work in our project: As the workspace can not safely be modified concurrently, they would each need their own workspace.

So the point of subagents is to research and condense results, to prevent context pollution of the parent. Given our previous conclusion, that we should compact earlier, subagents seem like an almost ideal solution.

Measuring their effectiveness is hard. The fact that they can not perform actual decompilation already makes them only half as useful. So the remaining question is, how efficient are they at researching.

Again, hard to measure, but what we can measure instead is how much redundant data they ingest, due to not sharing the context. This gives us at least a feeling of whether they are efficient or not.

There were 47 subagent runs in total. Across those 47 runs, I compared all big file reads (>= 5kB) and checked if there were redundancies:

36% of all distinct large files got read by more than one subagent. Weighted by how much was actually reread, it’s even worse: 54.7% of all the data subagents read from big files was pure duplication. The worst offender, `STATUS.md`, was read 28 times, by 21 different subagents (before we switched to GitHub issues).
That’s completely unnecessary, subagents shouldn’t need to read that file even once.

Importantly, the median time between a file’s duplicate reads by different subagents is 35 minutes. If the parent agent had done that research itself, it would have kept the file in its own context, and the vast majority of those redundant reads would never have happened.

So I decided to outright disable them.

### Did removing them decrease productivity?

| Period | Avg commits/day | 
|---|---|
| With Subagents | 243.9 | 
| Without Subagents | 311.1 | 

Using commits as productivity indicator is not necessarily accurate, but seeing an increased amount of commits per day doesn’t seem like productivity declined.

It’s still hard for me to quantify whether subagents are useful or not, even in general, outside this project. Yet, I feel like they can uncontrollably drive up token consumption (if tons of agents are spawned in parallel), which is enough of a reason for me to disable them.

## Does the Overseer actually catch anything?

The Overseer reviews every pushed commit. A question that I asked myself: Is it finding real bugs?

2177 of its Discord replies referenced a specific commit:

 

1172 of those messages were ambiguous and thus not classified. They were likely discussions with other agents, not distinct review messages.
However, 354 of those messages, roughly a third of all classified review messages, flagged a distinct bug. So it’s definitely catching issues and worth keeping.

## Model comparison: Opus vs Sonnet

Opus 5 and Sonnet 5 were each running almost half of the time.

| Model | Build attempts | Fail rate | Commits/h | 
|---|---|---|---|
| Sonnet 5 | 5,826 | 9.1% | 6.1 | 
| Opus 5 | 9,868 | 23.9% | 1.6 | 

What this suggests is that Opus causes much more build failures, spending much more time recompiling and fixing the code, resulting in way fewer commits per hour. This doesn’t necessarily show that Sonnet is better than Opus, it could just be that Sonnet picks simpler tasks, thus making fewer mistakes.

What makes sense is to combine these results with the reviews from the overseer.

### Overseer model comparison

I also tested using different models for the overseer.

Let’s look at bugs flagged per hour. Different worker models combined with different overseer models gives us the following constellations:

It looks like Opus is the better worker: Overall fewer bugs flagged per hour. However as we know, it’s also slower. If it commits less, it also makes fewer mistakes over a certain period of time. So let’s adjust the values to bugs flagged per 100 commits instead:

We can see both workers make almost the same amount of mistakes per commit. However, more importantly, Sonnet as overseer manages to flag more bugs than Opus.

**So, it’s the reviewer’s model that matters, not the worker’s!**

## Conclusion

Lots of interesting insights were discovered while evaluating the session logs.

Key takeaways for this project are:

- Sonnet is the better model: Cheaper, faster, and a better reviewer
- Early context compaction likely saves tokens
- Tweaking the setup matters and reducing build/test executions saves most of the time
- Soft instructions don’t really help, mechanical blockers, to enforce rules, are more effective
- An additional agent acting as reviewer is essential

We’ll see if the changes made here will have an impact and I will likely create at least one additional post when this experiment is done.
