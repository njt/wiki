---
url: https://markferree.substack.com/p/agent-orchestration-for-the-timid
date_fetched: 2026-07-05
backfilled: true
---

# Agent Orchestration for the Timid

### An exploration of AI agent orchestrators

Gastown was unavoidably everywhere I turned at the beginning of January.

Hovering daily between Steve’s stage 6 at work and stage 7 at home for my opensource projects I decided I would give Gastown a try before I fell into the trap of building my own ochestrator.

I love a little chaos in my life, I expected to fall in love with Gastown, my reaction was instead disgust and bewilderment. This tool seemed to abuse the model’s context for conflict resolution when a little bit of planning in a markdown file would have had better results. Duplication was an afterthought and sometimes even actively encouraged. The end result felt incredibly complex and token wasteful and unnecessary for the types of problems I had been successfully solving with a thoughtful combination of careful ai-engineering and vibe coding. I had been a Beads user since its initial release and after seeing the chaos of Gastown and fighting through a few painful and unnecessary feeling upgrades I started to rip Beads out of projects where I had been using it. Worrying that it had become (maybe always was?) a tightly coupled component of a bigger system rather than a useful standalone tool with its own vision.

Ok so not a Gastown convert, we get it, then whats a stage 7 Claude user to do when faced with the transition to stage 8? Inspired by a conversation with Claude about ai-codegen leading to duplication I decided to kick it like its 2021 and go shopping for an opensource Claude orchestration project rather than build my own from scratch.

As I would do with any research project these days I fed my bias for Agor over Gastown into Claude and after some back and forth got a good top 5 list of tools to try.

### Vibe Kanban

Install is super easy with nothing more than `npx vibe-kanban` works great if you already have your machine littered with node dependencies.

The npx command quickly and problem-free popped a local GUI with this fun message.

`Vibe Kanban runs AI coding agents with --dangerously-skip-permissions / --yolo by default, giving them unrestricted access to execute code and run commands on your system.`So much for a more conservative approach, but realistically if you want the agent to go Ralph mode you have to let it loop instead of letting it get stuck on silly things like removing your root directory.

I like the automatic environment detection and the large list of available agents is very promising given one of my dreams is to have frontier models and harnesses orchestrate less powerful models and harnesses to save time and money.

First impression is this feels like Jira, I’m manually typing up tasks and looks like I’m manually creating branches for said tasks too. One thing I loved about using Beads for project management was quick iteration and breaking off subtasks during my Claude conversations. I like Agor’s MCP self awareness for the exact same reasons managing worktrees and branches with aplomb.

Maybe Vibe Kanban has these features and I’m just not digging enough but I’m looking for love at first sight here so I’m going to save further exploration for another day.

Review: less buggy but less capable Agor, definitely more bug free and pleasant onboarding than any of the alternatives

### Conductor.build

Google form to join the Windows/Linux waitlist? This one was easy to remove from the list… next!

### Claude Squad

A cute name can go a long way I’m feeling good about Claude Squad.

Terminal only feels like a breath of fresh air. My main issue with Agor is my RSI not tolerating all the necessary clicky clicky to live in my own personal Notion inteface all day.

Seeing multi-agent support again if someone wants to give me a free Codex subscription I’ll definitely try this out.

Tmux as the way to separate things just like Gastown as a 10+ year tmux obsessive I love seeing all the attention tmux has been getting from ai-coders.

Learning curve feels steep here, I’m frozen staring at the command output wondering what to do next.

OH! If you open it from a git repository it just magically works.

Definitely glad I knew Tmux before I opened this, got stuck multiple times, feels like this is just a friendly Tmux wrapper around standard Claude sessions without a lot of extra overhead.

Yes, my personal Claude calls me Skippy, don’t judge.

I’m not sure having horizontal Claude sessions vs my normal vertical Claude sessions along with some keyboard shortcuts gets me a ton of value, but I’m also not pushing on any orchestration or subagent workflows either.

Committing is my final review and sanity check on AI output, I’m still holding onto that with all my strength. Not sure I want a tool that makes that a one character shortcut in my workflow.

Review: Too close to my personal workflow to be useful for me. My tmux-spawn Skill and slash command does a lot of this same heavy lifting in 100 lines of markdown.

### Claude-Flow

Easy to use (but slow and full of warnings) npx install command again.

Throwing “Enterprise AI Orchestration Platform” over the top of your chaotic NPM package looking github page AI generated logo with the website linking straight to a Discord does not give me Enteprise vibes.

I’m getting Gastown vibes with the talk of Queens and Swarms.

Three attempts before the install stuck, 7 different ways to install with 7 different nodejs tools also not giving me enterprise vibes.

Copy pasted commands straight from the README don’t work.

The task creation doesn’t feel like Jira, instead it feels like creating an SSL cert.

I like how everything is built into the same CLI tool, and the bi-directional integration with the Claude Code SDK and MCP with its own MCP feels powerful.

I started hitting bugs almost immediately my first task had no ID and then each subsuquent command needed that ID.

Review: putting this into the “vibe coded fever dream” category along with Gastown hopefully they have enough available tokens for Claude Flow to get itself in a working state.

### Taskmaster

And I thought Claude Flow had a lot of npm warnings…

Looks like they’ve got their own MCP, this should probably be table stakes for any ai-orchestration tool these days, I might actually like Agor’s MCP more than the Agor UI itself.

Oooh cute they have Hamsters instead of polecats, we’re overworking a lot of metaphors with these orchestrators.

Click Hamster and I’m sent to a browser window to sign up for a subscription, thats a big nope from me guess I’ll try Solo. A little clarity on where the free and open source ends and the commercial product begins here would be nice.

I like the support for multiple AI coding tools, orchestrating across tools is not something I’m seeing anyone else try to tackle.

Looks like just like Claude Flow all my task management is built into the same tool that orchestrates. It almost feels like they were working on a CLI task tool and bolted on AI capabilities if I had to guess.

Had to work my way one error to next through command line flags to attempt to create my first task feels more like using the AWS cli (thats not a complement).

Review: TaskMaster is off the list trying too hard to be commercial with lower overall quality and ease of entry than Vibe Kanban or Agor.

### Conclusion

Vibe Kanban is slick and easy to use, feels the most professional of the bunch and I might spend some more time with it as a potential Agor substitute. I might poke Claude-Flow with the same very long stick I use to poke Gastown in a few months. The last thing I’m going to do is pay for more chaos in my life sorry TaskMaster. Claude Squad just doesn’t do enough more than I’m able to do with skills and slash commands.

If anything after this exercise I’ll probably lean harder into Claude-centric features for orchestration than introduce the overhead of any of these orchestrators into my life. I’m getting plenty of value out of ai-centric coding today without letting an orchestrator take over.

actually i’ve been working on a similar idea. lightweight editor with LLM dev team, it’s been fun (https://x.com/guybedo/status/2016351710969790956?s=20)
