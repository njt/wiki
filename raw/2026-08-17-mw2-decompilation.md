---
url: https://web.archive.org/web/20260925153118/https://momo5502.com/posts/2026-08-17-mw2-decompilation/
date_fetched: 2026-10-11
---

As a little side project, I decided to put my Claude Max sub to good use and decompile Call of Duty: Modern Warfare 2 (2009).

The goal is to decompile the project to C++ code, as close to the original code as possible, but with adjustments where they make sense. That means additional changes for portability (32 & 64 bit, Linux, macOS, etc. support), security and stability improvements will be done.

This project is done with the help of RektInator, Future, st0rm and other members of the community. The repository is private for now.

## Setup

I’m using Claude Code CLI with a Max (20x) subscription. The workforce consists of 4 agents working together towards the common goal:

There are 3 worker agents, independently working on different subsystems of the game. All of them commit and push to the same branch of the repository.

An additional agent oversees the operation and reviews every pushed commit.

All agents currently use Sonnet 5.

Agents communicate via Discord. All agents have access to one channel and can both post and read all messages in there.

The overseer has access to an additional channel, where a GitHub webhook posts notifications about new commits that landed. That automatically triggers the agent’s review task.

An additional webhook posts CI failures in the shared channel, so agents get notified when something broke.

The agents themselves manage their tasks by using GitHub issues.

There are many interesting resources for decompilation: Leaked Xbox alpha builds with symbols, macOS ports with debug data, etc. that the agents use. They all have access to Ghidra and IDA Pro via MCP servers.

## Progress so far

The agents have been running non-stop for about 4 weeks now. They have produced almost 7,000 commits:

Roughly 5,588 of 16,324 functions have been decompiled so far: about 34%. The effective figure is higher, though: a good chunk of those 16k are third-party libraries, CRT code, and other things that will never need decompilation.

The game already launches:

The agents have burned through 199.8 billion tokens getting there.

However, there is also a big chunk of things that are missing or broken:

Launching a map is not working yet, despite many of the required subsystems being implemented. This will likely only be testable as soon as almost everything has been decompiled and wired up.

## What works well

Initially, progress tracking used a STATUS.md file in the repository. That file quickly grew into a 10 MB+ giant, instantly overflowing the context when trying to ingest it.

As a consequence, we switched to GitHub issues, which works great. Agents autonomously create, edit and close them.

 

Surprisingly, communication over Discord is also flawless.
Agents coordinate without any issues. They ignore messages not directed to them and only post messages when necessary to not needlessly fill the context of others with unrelated information, at least most of the time.

The Discord channel also enables us humans to interact with them, without me having to give the other participants access to the machines the agents are running on.

## However…

… there is a bunch of things that don’t work well, unfortunately.

We ask the agents to be concise in Discord. They are, for exactly 5 minutes, then start writing walls of text again…

One message like this might be fine, but agents actively communicate in that Discord channel. Many of those messages are directed to us humans, status updates, etc. Nobody wants to read that much text.

Another pain point is local test execution. For some reason, agents are overly careful. On every little change, they feel the urge to execute the entire test suite locally. Local test execution is almost always green. Therefore every successful test run means about 4 minutes of time that could have been spent decompiling code instead. They were repeatedly told to never execute tests locally. We have CI for that. If a commit causes a failure, there will be an almost instant notification from GitHub via Discord. Failures will never go unnoticed. Yet, they don’t really care.

One thing that is becoming a problem is that agents are afraid of starting bigger issues. They favor quick wins over larger subsystems. The moment they stumble upon bigger blockers, they file an issue, defer the problem and move to something smaller. Initially, that was fine. Now that low-hanging fruit is mostly gone, this leads to agents being idle at times.

From time to time, agents start refusing to work, because they think they’re running out of context. Obviously, they don’t. They have no idea how many tokens are in their context, and even if they did, that’s something the harness takes care of. Auto compaction will kick in when there is an overflow.

 

Sadly, GitHub also has had issues every other day so far. That makes issue tracking and CI relatively unreliable.
I hope they manage to improve their uptime, otherwise switching to an alternative makes sense.

## What I have learned

Giving the agents too many tasks at once just doesn’t work over a long period of time. Telling them to decompile, modernize C++ code and improve portability all at once just doesn’t work. They either eventually forget one of those things after the 10th compaction, or start making mistakes.

Pure mechanical decompilation first, and then modernization/portability afterwards is the way to go.

I ran Opus 5 for the first 2 weeks and eventually decided to switch to Sonnet midway. It is slightly dumber, yet there is no noticeable difference. As far as I can tell, Sonnet is making the same mistakes as Opus and gets distracted just as easily. After all, decompilation is a relatively easy task. It’s just an MCP call to IDA and then pasting that into a CPP file. Sonnet works fine for that.

To improve their focus on the rules we set, I added PostCompact hooks that reinject a set of rules into the context. This seems to work to some extent, they properly manage GitHub issues over longer periods of time and don’t forget to post updates on Discord that way. But staying concise with their messages or omitting redundant code comments is just something they don’t respect. My best guess is that yapping, both in code and in messages, was just deeply embedded during their training.

## Summary

Although not everything went well, this experiment has still been a success. Overall decompilation works very well. The code will need a round or two of refactoring, but I think it needed that regardless. MW2 is an old game with lots of potential for improvement.

It’s very interesting that a different class of problems starts to arise when working on multi-day and multi-week sessions. I think most of the issues can be attributed to compaction and thus agents forgetting what matters over time. Previously, I have only ever had hour-long sessions, where compaction still causes harm, but not to the same extent.

The project is far from done. We’ll see where things are going and how it evolves. I might post updates when something interesting happens.

At this point, I’m openly inviting you, the reader, to give feedback. If you have ideas of improvement for the issues I have had, feel free to share your experience with me via my socials.

Click here for the next part of this series.
