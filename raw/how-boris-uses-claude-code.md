---
url: https://threadreaderapp.com/thread/2007179832300581177.html
date_fetched: 2026-07-05
backfilled: true
---

I'm Boris and I created Claude Code. Lots of people have asked how I use Claude Code, so I wanted to show off my setup a bit.

My setup might be surprisingly vanilla! Claude Code works great out of the box, so I personally don't customize it much. There is no one correct way to use Claude Code: we intentionally build it in a way that you can use it, customize it, and hack it however you like. Each person on the Claude Code team uses it very differently.

So, here goes.

1/ I run 5 Claudes in parallel in my terminal. I number my tabs 1-5, and use system notifications to know when a Claude needs input code.claude.com/docs/en/termin…

2/ I also run 5-10 Claudes on claude.ai/code, in parallel with my local Claudes. As I code in my terminal, I will often hand off local sessions to web (using &), or manually kick off sessions in Chrome, and sometimes I will --teleport back and forth. I also start a few sessions from my phone (from the Claude iOS app) every morning and throughout the day, and check in on them later.

3/ I use Opus 4.5 with thinking for everything. It's the best coding model I've ever used, and even though it's bigger & slower than Sonnet, since you have to steer it less and it's better at tool use, it is almost always faster than using a smaller model in the end.

4/ Our team shares a single CLAUDE.md for the Claude Code repo. We check it into git, and the whole team contributes multiple times a week. Anytime we see Claude do something incorrectly we add it to the CLAUDE.md, so Claude knows not to do it next time.

Other teams maintain their own CLAUDE.md's. It is each team's job to keep theirs up to date.

5/ During code review, I will often tag @.claude on my coworkers' PRs to add something to the as part of the PR. We use the Claude Code Github action (/install-github-action) for this. It's our version of @danshipper's Compounding Engineering CLAUDE.md

6/ Most sessions start in Plan mode (shift+tab twice). If my goal is to write a Pull Request, I will use Plan mode, and go back and forth with Claude until I like its plan. From there, I switch into auto-accept edits mode and Claude can usually 1-shot it. A good plan is really important!

7/ I use slash commands for every "inner loop" workflow that I end up doing many times a day. This saves me from repeated prompting, and makes it so Claude can use these workflows, too. Commands are checked into git and live in .claude/commands/.

For example, Claude and I use a /commit-push-pr slash command dozens of times every day. The command uses inline bash to pre-compute git status and a few other pieces of info to make the command run quickly and avoid back-and-forth with the model (code.claude.com/docs/en/slash-…)

8/ I use a few subagents regularly: code-simplifier simplifies the code after Claude is done working, verify-app has detailed instructions for testing Claude Code end to end, and so on. Similar to slash commands, I think of subagents as automating the most common workflows that I do for most PRs.

9/ We use a PostToolUse hook to format Claude's code. Claude usually generates well-formatted code out of the box, and the hook handles the last 10% to avoid formatting errors in CI later.

10/ I don't use --dangerously-skip-permissions. Instead, I use /permissions to pre-allow common bash commands that I know are safe in my environment, to avoid unnecessary permission prompts. Most of these are checked into .claude/settings.json and shared with the team.

11/ Claude Code uses all my tools for me. It often searches and posts to Slack (via the MCP server), runs BigQuery queries to answer analytics questions (using bq CLI), grabs error logs from Sentry, etc. The Slack MCP configuration is checked into our .mcp.json and shared with the team.

12/ For very long-running tasks, I will either (a) prompt Claude to verify its work with a background agent when it's done, (b) use an agent Stop hook to do that more deterministically, or (c) use the ralph-wiggum plugin (originally dreamt up by @GeoffreyHuntley). I will also use either --permission-mode=dontAsk or --dangerously-skip-permissions in a sandbox to avoid permission prompts for the session, so Claude can cook without being blocked on me.

13/ A final tip: probably the most important thing to get great results out of Claude Code -- give Claude a way to verify its work. If Claude has that feedback loop, it will 2-3x the quality of the final result.

Claude tests every single change I land to claude.ai/code using the Claude Chrome extension. It opens a browser, tests the UI, and iterates until the code works and the UX feels good.

Verification looks different for each domain. It might be as simple as running a bash command, or running a test suite, or testing the app in a browser or phone simulator. Make sure to invest in making this rock-solid.

We're launching Claude Tag today. Tag Claude into Slack and it works in channel with you. It’s proactive, multiplayer, with its own identity and memory.

But it’s not just a bot in Slack. Over the last few months, it’s totally changed how we use Claude

This is the start of Claude Everywhere. It’s Claude Code under the hood so it’s just as good at writing code. 65% of our product team’s new code is created by our internal version of Claude Tag

Claude is really proactive with Claude Tag. You don’t need to prompt it to do work, it can do work proactively based on your instructions. It has excellent memory and access to your data, so it can behave differently per channel

Dogfooding Opus 4.7 the last few weeks, I've been feeling incredibly productive. Sharing a few tips to get more out of 4.7 🧵

1/ Auto mode = no more permission prompts

Opus 4.7 loves doing complex, long-running tasks like deep research, refactoring code, building complex features, iterating until it hits a performance benchmark.

In the past, you either had to babysit the model while it did these sorts of long tasks, our use --dangerously-skip-permissions.

We recently rolled out auto mode as a safer alternative. In this mode, permission prompts are routed to a model-based classifier to decide whether the command is safe to run. If it's safe, it's auto-approved.

This means no more babysitting while the model runs. More than that, it means you can run more Claudes in parallel. Once a Claude is cooking, you can switch focus to the next Claude.

Auto mode is now available for Opus 4.7 for Max, Teams, and Enterprise users. Shift-tab to enter auto mode in the CLI, or choose it in the dropdown in Desktop or VSCode.

2/ The new /fewer-permission-prompts skill

We've also released a new /fewer-permission-prompts skill. It scans through your session history to find common bash and MCP commands that are safe but caused repeated permission prompts.

It then recommends a list of commands to add to your permissions allowlist.

Use this to tune up your permissions and avoid unnecessary permission prompts, especially if you don't use auto mode.

Just got a nice DM from a big enterprise customer using Claude Code in one of the world's biggest codebases

Here's how we made @-mentions 3x faster in large enterprise codebases 🧵

To power file @-mentions in Claude Code, we used to use a custom Rust-based file indexer that we invoked from Claude Code via NAPI. It was pretty fast.

But in big codebases, it would fall over.

So we introduced a new fileSuggestion setting in Claude Code, so customers with large codebases could plug in their custom code index (using SourceGraph, an in-house index, etc.).

Starting tomorrow at 12pm PT, Claude subscriptions will no longer cover usage on third-party tools like OpenClaw.

You can still use these tools with your Claude login via extra usage bundles (now available at a discount), or with a Claude API key.

We’ve been working hard to meet the increase in demand for Claude, and our subscriptions weren't built for the usage patterns of these third-party tools. Capacity is a resource we manage thoughtfully and we are prioritizing our customers using our products and API.

Subscribers get a one-time credit equal to your monthly plan cost. If you need more, you can now buy discounted usage bundles. To request a full refund, look for a link in your email tomorrow. support.claude.com/en/articles/13…

Today we're excited to announce NO_FLICKER mode for Claude Code in the terminal

It uses an experimental new renderer that we're excited about. The renderer is early and has tradeoffs, but already we've found that most internal users prefer it over the old renderer. It also supports mouse events (yes, in a terminal).

Some of the upsides:
- No more flickering
- No more jumping
- Constant memory and CPU usage as the conversation grows
- Mouse support! You can now click to move your cursor within the input box. Some other UI elements are also clickable now.
- Nicer selection behavior. eg. when you select code, we no longer include line numbers and UI elements in the selection

There's also downsides:
- Native cmd-f doesn't work. Instead, hit ctrl+o then / to search the transcript (or, use ctrl-r for reverse search)
- Native copy-paste doesn't work. Instead, we copy to the clipboard by default when you make a selection. You can configure this in your settings.json to use ctrl+c instead, if you prefer.
- Scrolling gravity varies by device, and we are in the process of tuning the physics to make it feel good on every terminal

I wanted to share a bunch of my favorite hidden and under-utilized features in Claude Code. I'll focus on the ones I use the most.

Here goes.

1/ Did you know Claude Code has a mobile app?

Personally, I write a lot of my code from the iOS app. It's a convenient way to make changes without opening a laptop.

Download the Claude app for iOS/Android > Code tab on the left.

2/ Move sessions back and forth between mobile/web/desktop and terminal

Run "claude --teleport" or /teleport to continue a cloud session on your machine.

Or run /remote-control to control a locally running session from your phone/web. Personally, I have "Enable Remote Control for all sessions" set in my /config.
