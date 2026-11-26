---
url: https://ai.engineer/code/2026/talks/online-ayush-cli
date_fetched: 2026-11-26
---

Machine-generated source transcript; recognition errors remain. Timestamps and subtitles use the selected source recording’s clock, not clip-relative time. Section links use cited source moments; sections without a verified moment return to the talk start.

1:46:46Hi, I am Ayush.

1:46:47I built a CLI that lets your agent reuse context from your past session across tools.

1:46:52While improving it, I realized why my agent secretly hated my CLI and what you can do about yours.

1:47:01CloudCode wanted one setting from a docs page, so it downloaded the whole page,

1:47:06grabbed it, and kept the result at 40 lines. Codex wanted to know how many dependencies

1:47:12were vulnerable. It asked the whole JSON report and kept the last 40 lines

1:47:17where this report puts the totals.

1:47:20Cursor ran a test suite, found one failure, and kept the last 80 lines.

1:47:25All three cuts kept the information shown here, each one encoded in an assumption about

1:47:30where the answer sits in the output.

1:47:32That can hide friction from the people who build these tools.

1:47:35One pipe tells you very little, the same pipe written again and again can point to something

1:47:40your tool is missing.

1:47:42These are real calls from my own transcripts and the cloud code one was the agent reviewing

1:47:46this talk and it still couldn't help itself. Agents can work around friction without filing

1:47:53a bug report. They write a little program around your program. In June, Sanity, the

1:48:00content platform, wrote up their MCP server. More than 20,000 agents, about 3 million tool

1:48:06calls since launch, and not a single bug report. So they shipped a GIF feedback tool. Once

1:48:11the agent had somewhere to complain, they started expressing what's wrong. But you

1:48:15You don't need a feedback tool to start.

1:48:18Feedback is already sitting somewhere in your transcripts.

1:48:23I extracted about 20,000 agent calls from my same transcripts.

1:48:2714 CLIs appear in that sample, git, github-cli, cargo, curl, npm-docker, and others.

1:48:33A quarter fed output into filters such as head, grab, tail, python, sort, or wc.

1:48:39For read-heavy commands, it's more than half.

1:48:42Git show and cargo test 62% each.

1:48:44Some harnesses pipe far less, codex for example, lets the agent set an output limit on each call.

1:48:53And pipes don't only cut noise. In bash, without pipe fail, a pipeline returns the exit code of its last command.

1:49:01So let's say you pipe cargo test into head. It can report success even when the test failed.

1:49:07Here's one example on the screen, a compile error and a green dot.

1:49:13The status was wrong, but the agent could still read the error.

1:49:18A pipe can also cut the error itself, which is worse.

1:49:22In a cloud code issue on GitHub, a user reported it a script through a reference error and

1:49:27exited with an error code.

1:49:28Tails showed three harmless lines and Cloud reported success.

1:49:32The error never reached the model.

1:49:34So my advice would be to fail loudly.

1:49:37when a tool fails, make the very last lines a short summary and a part to the full log.

1:49:45I learned all this while tinkering with the CLI I built. It's a small tool that lets coding agents

1:49:51search their own past sessions across cloud code, codex cursor. It shifts with a skill so the agent

1:49:57can decide when and how to call it.

1:50:00Agents are the users that I'm studying. My October audit recorded about 1,000 calls and that's my small case study.

1:50:08The method works on anything your agent calls. Just look at what they do right after the call.

1:50:1580%

1:50:174 out of 5 times an agent opened an old session with my tool. The output went straight into Crap, Sad, Head, Tail or Python.

1:50:25I call that the pipe text.

1:50:27A pipe keeps junk out of context, and that's fine.

1:50:30The text is that the agent writing that same cutdown command by hand every single time.

1:50:36It's small each time, and it mostly works, but each one is another guess about where the answer is.

1:50:43The fair objection comes from a hacker news thread about designing CLS for agent that I was reading.

1:50:49The command says to leave it to the agents to choose how much output to read.

1:50:54agent can pipe through grab or head perfectly well. Well, they can and for one-off they should.

1:51:01I'm not against that but across those everyday CLIs I found 83% of pipes fell into the pattern

1:51:07seen at least 10 times the same CLI subcommand and pipe tools. The arguments past could differ

1:51:13but the agent keep asking for the same things. I understand the output they're cutting is

1:51:20probably big. I opened one long session with my tool two ways. The whole thing, 1.2 million

1:51:27characters and that's why the agent spy. Just the part around the search match was about 3,000

1:51:32characters. Harness is do-cap output but a size gap does not identify the relevant messages.

1:51:39Selection flags on the other hand let the agent tell your tool which part it wants.

1:51:45So based on my experience and improvements I made to my own tool, I'll give you three

1:51:50examples.

1:51:51Read the left column as the bug reports.

1:51:54So starting with the first one, filtering with RG and said means let me pick the part

1:51:58I want.

1:51:59So my read command view can now written just the part around a match, only the matching

1:52:05messages or the last few with a size gap.

1:52:09Second example would be Python over JSON.

1:52:12So Python over JSON means give me fewer fields. So search got a compact mode one line per hit about six times smaller than JSON with the same results.

1:52:23Inspect command in chat history summarizes a session.

1:52:26About a third of inspect calls came from agent looping over several sessions by hand as you can see with the for loop.

1:52:33That means let me ask several about several at once and now it's one call.

1:52:38As you can see, my roadmap was in the commands around my command and I had to learn from that.

1:52:47InfraCost, the CloudCost tool, wrote up the same discovery in May.

1:52:50Their agents were writing JQ and Python pipelines over JSON dumps, so they moved the filtering into the CLI and added a compact format and a skill to teach it.

1:52:59Plain Cloud with none of that got 0 of 6 hard questions right in the testing and with it,

1:53:056 of 6 and cost across the benchmark also fell 41%.

1:53:09On Hacker News, the person who wrote that post said that reading those agent logs felt

1:53:14like watching usability tests and I couldn't agree more.

1:53:19This part might sound trivial and obvious.

1:53:22Once it's in the CLI, teach your agent it's there with the skill file or the help flag.

1:53:26But agents take their teaching literally. I made some changes that turned my deep flag

1:53:32into a no-op. I left the skill telling the agents to use it to see what they do,

1:53:37would they find out or not. 95 searches out of 95 still passed it. After that,

1:53:44two sessions loaded a rewritten skill and that no longer told them to use it. None of the searches

1:53:50passed deep flag and both tried new flags on their first read. So your skill.md, your help

1:53:56text, your error messages are the interface. That also means stale dogs are bugs because

1:54:01your agent will keep following them. Once the new flags reach agents, nearly every search used

1:54:09compact output and nearly every reduced the new flags. Piped read fell from 82% to about a quarter.

1:54:22You can get real levels and test cases for your tool easily.

1:54:25If you use it every day, they are already in your transcripts.

1:54:28The commands agent really ran are the test cases and what agent did immediately next

1:54:32is a rough answer key.

1:54:34Replay those commands through your old build and your new one.

1:54:37If you change the ranking, check that result the agent went on to use still comes out on

1:54:44the top or not.

1:54:44If you change an output format, check that nothing went missing.

1:54:48If you change the default, see which commands now behave differently.

1:54:52And when an agent read a call with different arguments, you have probably found a failing

1:54:57case.

1:54:57The answer key is only a rough guess at what the agent wanted.

1:55:01Keep a few cases back and use those to score your fix.

1:55:06So after analyzing 20,000 invocation, if I were to start all over again, this is what

1:55:12I would build first.

1:55:1355% of the measured pipelines used only head, tail, or set downstream.

1:55:19That points to demand for smaller slices.

1:55:22So ship ranges and limits before anything fancy.

1:55:25A flag only helps if agents reach for it.

1:55:29GitHub CLI's list command have a limit flag.

1:55:31An agent used it 81% of the time.

1:55:34The counter example would be cargo.

1:55:36For cargo test, quiet mode,

1:55:38prints a character per test instead of a line and still reports failures.

1:55:43Agent passed it on about one cargo run in 10 and piped instead.

1:55:48So teach the flag in your help test and your skill.

1:55:52And make big outputs print less by default and say how to get more.

1:55:57For example, end with something like showing 20 of 300, add offset flag 20 for the next wage.

1:56:05One way to ship several of these at once is an agent mode.

1:56:09Your CLI notices an agent is calling it and prints differently.

1:56:12Agents like Claudecodes set an environment variable in their cells so your tool can spot them.

1:56:18DuckDB shipped one. At a terminal you get a nicely drawn table but an agent gets a plain one that says how many rows were left out and how to get them.

1:56:27A huge query stops early and says so and errors come back as JSON because agents read them better.

1:56:34It's the same list as we have been just through and it cut the output model read by 59%.

1:56:39However, the tokens saved were nearly 0.5% of all the tokens the model read.

1:56:45So cost and runtime didn't improve.

1:56:48If you remember, InfraCost agents started getting answers right.

1:56:52DougDBs were already there.

1:56:54So measure turns and write answers, not just tokens.

1:57:00might be thinking, I'll ask my agent to do all of this.

1:57:04They are not that good at this yet.

1:57:06At least not one-shotting it.

1:57:08The design is where it goes wrong.

1:57:10Ask an agent to build a tool for agent.

1:57:12It probably pictures a person and then bolts the agent on.

1:57:16It copies most common patterns it has seen, and most of those are poor.

1:57:20A study of official MCP servers found most just rep an existing API.

1:57:26Another checked tool descriptions from real MCP servers and flagged issues in 97% of them.

1:57:33Docs have the same gap.

1:57:34researchers at each year Zurich tested context files written by agents. They didn't help tasks

1:57:40succeed, cost more to run, and did worse than files written by people. Agents are already good

1:57:46at other half of this talk. Give one transcript and a test it can run, and it can find what's

1:57:52wrong and improve the tool, sometimes past what people wrote by hand. And Tropic reported that

1:57:57on its own internal tools as well. So you decide what the tool should be and let the agent check

1:58:03against what really happened. You've seen the evidence for each of these so I

1:58:08won't read them. Take a moment to absorb these or screenshot.

1:58:18To find your own list, backup your transcript first because Cloud Code can

1:58:23delete inactive once after 30 days. Then search for them for a tool's name

1:58:29followed by a pipe and other similar patterns.

1:58:31If you ship an MCP server, look for agent calling your tool again,

1:58:36straight away with different arguments.

1:58:40If you remember one thing from this talk,

1:58:43if you are building a tool for your agent,

1:58:45your agent transcripts are the product analytics you probably aren't reading.

1:58:51The workaround is the bug report, so go read your transcripts.

1:58:55And if you need any help doing that, check out Chat History on Cargo or GitHub.

1:59:00It can do this for you as well.

1:59:02Thank you.
