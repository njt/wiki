---
url: https://blog.jonudell.net/2026/06/17/vibe-coding-as-a-team-sport/
date_fetched: 2026-07-05
backfilled: true
---

In Working With Intelligent Machines, written at the beginning of my AI-assisted coding journey, I quoted from Garry Kasparov’s *The Chess Master and the Computer*.

The winner was revealed to be not a grandmaster with a state-of-the-art PC but a pair of amateur American chess players using three computers at the same time. Their skill at manipulating and “coaching” their computers to look very deeply into positions effectively counteracted the superior chess understanding of their grandmaster opponents and the greater computational power of other participants. Weak human + machine + better process was superior to a strong computer alone and, more remarkably, superior to a strong human + machine + inferior process.


Bram, the tool I’m building to support that kind of teamwork, puts a UI next to Claude Code and Codex and guides them through a workflow that’s anchored to git for version control and GitHub for collaboration.

## A UI companion for the terminal

Here’s a picture of of me using Bram to build a standalone voice transcription app. Bram itself is a desktop app that wires together a terminal where you run Claude Code or Codex (on the left), a companion UI for them (bottom right), and the app you are developing (top right).

At the moment this screenshot was captured I was testing the first iteration of my transcription app, and discussing with Claude Code how the app will manage its own the Whisper server.

Bram’s UI puts a microphone next to several input boxes so you can capture voice, and it uses Whisper to transcribe what you say. For me this is transformative. I’ve long struggled with repetitive stress and reducing my keystroke load really helps. Now, as I use Bram to develop Bram — as well as the apps I build with it — I rarely have to type.

The UI echoes agent responses more readably than they appear in the terminal. It reports recent tool uses as a compact list of links that you can open to see tool calls and results. And when you paste a screenshot, it displays the image so you can see what you and the agent are talking about. (When you paste an image into the terminal, it just appears as [Image #1].)

## Guardrails for vibe coders

The workflow brings a few layers of structure to the conversation that you’re having with agents. The guardrails are optional, but by default Bram wants you to put items on a worklist. In the software world these are often called stories on the backlog, but I’m not assuming that someone who’s using Bram will be familiar with that tradition. LLMs are bringing a lot of people to coding who have never coded before, and have never touched a terminal or git or GitHub. For them, Bram aims to be an on-ramp to these disciplines.

You ask Bram to file a new worklist item by giving it a brief description of what you want to do. Or you choose an open issue from the GitHub repository that Bram runs in, and it builds a worklist item based on that issue. The item shown in the screenshot is *whisper-server-lifecycle*.

However you ask Bram to create it, the new item appears with a before and after section. The before section describes the current state of play. The after section says how things will be when the plan becomes code. It lists options considered, justifies the chosen one, cites prior art in the code or in related GitHub issues, and outlines ways to verify that the changes yield the desired result.

Now the item waits at the *To-Apply* gate, one of two approval gates in the workflow. For either, your choices are Approve, Iterate, or Drop. If you Approve at the *To-Apply* gate, Bram implements the plan and advances to the next approval gate, *To-Commit*. But you might want to click the Iterate button and refine the plan document. You can do this as much as you want.

## Flexible workflow

Often, as you iterate, you and/or your agent will realize that the current item touches other parts of the system, or suggests new ideas, or raises concerns that you haven’t considered. You can ask Bram to capture these tangents as new worklist items or GitHub issues.

Sometimes after you iterate for a while you realize that the item just doesn’t make sense — maybe not now, maybe never. Use the Drop button to remove the item from the worklist. The history is retained; you and agents can review and search that history.

Another way to preserve an item you’re not ready to take forward: ask Bram to promote it to a GitHub issue that carries the plan of record. You can bring it back later as a new issue-derived item.

During each iterate cycle you’re typing (or in my case speaking) into an input box where you can attach one or more screenshots — incredibly helpful if you’re building UI.

When ready to advance an item, click Approve. Up to this point Bram has made no changes to tracked files in the repository. Now it begins to do so, constrained by a self-created list of the files it expects to touch.

At this point you’re in the familiar loop where Claude Code is shenaniganing and wibbling or Codex is doing its equivalent. Perhaps, depending on your permission settings, they are prompting to make tool calls. Bram’s UI helps here in a couple of ways. When a tool asks permission to use an *awk* command with a long gnarly string of arguments, the command is easier to read than in the terminal. (Caveat: accurate parsing of the menus presented by the Claude Code and Codex TUIs — text user interfaces — is a work in progress!) And when the agent proposes a change, the diff is easier to read than in the terminal. But the terminal is right there and I tend to keep eye on it, the companion UI just gives you more to see and do. While an agent is thinking, and you are waiting, you can open and review the item’s plan. You can switch over to the Issues tab and review what’s going on there. You can review tool calls to see more clearly what’s happening under the hood. You can create and iterate new worklist items.

## The team dimension

Implementing the review and approval lifecycle in a way that’s reliable, and works identically for Claude Code and Codex, has proven to be an interesting challenge. For me it’s not an either/or thing. I’ve always found it valuable to consult multiple LLMs and play one off against the other. Inside Bram, quite often, I switch from Claude Code to Codex, or vice versa, and ask one to weigh in on a worklist item, commit, or issue that was touched by the other.

That’s one aspect of the kind of teamwork that I’ve often talked about in my series of posts on working with LLMs. I regard them as a team of assistants and, until recently, I would often copy a transcript from one and paste it into the other. Now, with Bram, agents can see more than the code and documentation in the repository. They can see and react to plans on the worklist and discussion in related GitHub issues. This is useful even if you’re operating as a solo developer, because information that would otherwise be squirreled away in hidden files seen only by one agent or another are now visible to, and searchable by, all of them. It leads to amusing interactions:

“Hey Claude, grab that evidence from the log and post it along with a comment on issue 185 so Codex can weigh in.”

“Hey Codex, look at what Claude said, what’s your take?”

Bram enables this by guiding agents to the *gh* commands that can not only create and edit issues but also post and edit comments on issues. When you work this way your team now includes not only you and your agents but also your human team members and their agents. Bram is a solo project right now, but when I am working on XMLUI (which powers the Bram UI) this communication is directed to the whole team. To clarify who’s talking, Bram encourages agents to introduce themselves: “This is Jon’s Codex speaking, Jon asked me to weigh in”.

As the person directing the agents, you are now in a position to curate outboard context that’s available to the whole team. If you’ve worked with agents, you know that they can often be quite verbose. You get to decide how much to include. Usually I tell agents to begin with an executive summary for the benefit of people, but include full details for the benefit of other agents who will happily read and absorb this additional context.

## Just enough ceremony

In How to make best use of git and GitHub for AI-assisted software development I showed how agents can wield command-line tools like *git* and *gh* on our behalf. If you use these tools regularly you may not appreciate how much tacit knowledge you’ve acquired. Without LLM help no newbie would stand a chance. But even veterans, if they are honest, will admit that these tools are byzantine and cumbersome, and that it’s a great relief to use them fluently without having to remember command syntax.

My collaborator on this project, Andrew Schulman, is using Bram to develop a tool for code analysis. When I showed him that first post about git and Github he said: “You’re underselling the workflow.” It’s early days, and things are evolving quickly, but we are both certain that we are far more effective with this workflow than without it. LLMs. Bram is already complex, Andrew’s code exam is even more complex. With Bram we feel we are bringing order to the chaos of vibe coding and managing complexity that we otherwise would not be able to handle.

I’ll let Andrew speak for himself but for me it’s about having just enough ceremony for the task at hand. If you’re doing a small thing, like changing one line of code or tweaking a piece of documentation, you can tell Bram to skip the worklist and roll your tweak into an open item or an unpushed commit. If it’s a bigger thing, you want — or you should want, and Bram wants you to have — more structure. The worklist enforces ceremony in a local and transient way, GitHub enforces it in a shared and permanent way, and work can flow in both directions as needed. For people and their agents, this is how vibe coding becomes a team sport.

I’ve become shockingly more productive working with Bram. For a brief description and a screenshot from working with Bram on forthcoming CodeExam, see my home page at https://www.softwarelitigationconsulting.com/ .
