---
url: https://gist.github.com/8fc5d0c9cd23ee37abb8bd50351b3c1e
source_url: https://www.youtube.com/watch?v=dUMsFQ8y3gM
title: Lessons from Building Cursor
channel: ByteByteGo
speaker: Unnamed Cursor team member (Speaker B)
date_fetched: 2026-07-04
duration: 25m 33s
topics:
  - coding-agents-and-frameworks
  - ai-research-and-models
---

# Lessons from Building Cursor

Interview on ByteByteGo with a Cursor team member (Speaker B, joined by interviewers A and C). Covers Composer 1.5 model training, RL infrastructure at scale, context window solutions, cloud agents, and what "coding got solved" actually means.

## Summary

### Model-Building as Product-Building

Cursor trains its own models (Composer 1.5, positioned "somewhere between Sonnet 4.5 and Opus 4.5 in capability") because certain capabilities must be "built into the model" — they can't be prompted into existence. Semantic search across large codebases is the prime example: an RL-trained model finds what it needs in 1–3 queries rather than tens of greps. Recursive subagents and self-summarization are similarly only learnable through RL, not prompting. The model has to experience millions of sandbox sessions to internalize effective tool use patterns.

### RL Infrastructure Is a New Kind of Hard

Running RL training at Cursor's scale requires "millions of sandboxes" and "100 million plus of CPU compute per year." This scale forces in-house orchestration — "you can't actually buy the infrastructure from anyone." Long-running agents (minutes to days) break traditional RPC assumptions. The speaker points to Temporal and Restate as workflow engines suited to this paradigm, but notes that deployments for 12-hour-running agents are uniquely difficult.

### Context Windows Are Solved Through RL

Instead of trying to prompt the model to produce good summaries, RL "forces the model to produce" summaries that are genuinely useful to its future self. The model also learns to dump old conversations to files and grep them later. "You don't need a trick" — the incentive structure of RL solves the framing problem naturally. The model learns what information its future self will actually need, because RL rewards correctness on downstream tasks.

### Cloud Agents Need a Step Change, Not UI Tweaks

Cloud agents today are "a worse version of the local agent" — slower to boot, harder to review diffs. The speaker argues that to grow cloud agent compute from ~1% to ~90% requires a step change, not incremental improvements: "small tweaks never really get you factors of 1000." The key insight is that the model must test its own code and prove correctness. "It should feel like the model wrote the code and it should be the model's responsibility to figure out if it's correct or incorrect."

### The Devex Problem for AI

Models "silently degrade" when services aren't started in the right order — unlike humans, they don't complain. The speaker predicts "companies in the future will have some devex teams" that document boot-up procedures specifically for AI agents. This is a new category of infrastructure: runbooks not for onboarding humans, but for making environments legible to models.

### Model Routing Is a UX Problem

Cursor avoids routing because "a lot of people get very confused." The ideal: "you just press enter, you forget about it" — like Google. The speaker sees routing as a temporary crutch, not a permanent architecture.

### "Coding Got Solved in Six Months"

Presented as "a boring fact about the world, not an inflammatory claim." Between March/April and December of some recent year, "the best engineers I know are not writing code by hand anymore." The next capability jump: engineers adopting a "managerial instinct" — becoming more like managers overseeing agents than coders typing syntax. The speaker predicts "one to two capability jumps" every half year.

### The Browser Experiment

The speaker describes watching a model build a functional browser over three days with "three or four thousand commits" — and realizing "for the first time I was thinking, I can't do this." This led to the vision of "self driving code base": allocate a budget, and the codebase manages security, tech debt, bugs, and features autonomously.

### Advice for Engineers

"If your code base will stay for many years, review every line of code. If your code base is for a weekend, who cares?" The nuance: review rigor scales with expected lifetime. But also: the speaker claims most engineers "actually enjoy coding more now" because "boring aspects like debugging for hours" have diminished.

### Pithy Quotes

- "It should feel like the model wrote the code and it should be the model's responsibility to figure out if it's correct or incorrect."
- "You can't really grow anything by a factor of 1000 by just tweaking the UI."
- "The best engineers I know are not writing code by hand anymore."
- "If your code base will stay for many years, review every line of code. If your code base is for a weekend, who cares?"
- "Those things can only be learned during rl."
- "In some ways, coding got solved in six months" — "a boring fact about the world, not" an inflammatory claim.
- "You don't need a trick" — on context windows.

### Tools, Practices, and Methodologies

- **Composer 1.5**: Cursor's in-house model, RL-trained, between Sonnet 4.5 and Opus 4.5 capability
- **RL for Tool-Use**: Semantic search, recursive subagents, self-summarization — capabilities only learnable through RL
- **Self-Summarization with File Dumps**: Model learns to write files and grep them later
- **Temporal/Restate**: Workflow engines suited to long-running agent orchestration
- **Cloud Agents with Built-in Testing**: The step change — model proves its own code is correct
- **Devex Runbooks for Models**: Documentation of boot-up procedures for AI agents
- **Managerial Workflows**: Engineers becoming managers of agent teams
- **Self-Driving Codebases**: Allocate budget, codebase manages itself

### Unanswered Questions and Omissions

- Specific RL algorithms not discussed (PPO? DPO? Something custom?)
- How the model actually "proves" correctness
- Why routing doesn't work and what the alternative is
- Concrete lessons from the browser experiment about agent organization
- Self-driving codebase guardrails — what prevents runaway changes?
- Cost and environmental impact of millions of sandboxes
- What "coding got solved" actually means — which coding? All coding? Web apps? Systems?
- Concrete advice for engineers transitioning to "managerial instinct"
- Job displacement and ethics — entirely unaddressed
- Whether the "review every line" advice applies when agents write thousands of lines per session
