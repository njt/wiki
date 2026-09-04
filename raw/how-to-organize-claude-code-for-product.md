---
url: https://www.theaithinker.com/p/how-to-organize-claude-code-for-product
date_fetched: 2026-09-04
---

My Claude Code workspace for my product job is a folder with almost no code in it. It holds a context folder with everything about the company: the product, the users, the competitors, how I like to work. One folder per project, with the tasks separated from the deliverables. A set of skills that each do one recurring piece of work: the status update, the PRD review, the interview synthesis. When I give feedback once, it lands in a file with a note on why, and I never give that feedback again. **I didn’t design this system. I arrived at it, mostly by getting it wrong first.**

Getting it wrong taught me the lesson that runs this whole article. **Past the basics, your results in Claude Code stop depending on how well you prompt and start depending on how well you file.** A good prompt improves one session; a good file improves every session after it. The prompt-tuning advice you’ve already read stops helping at exactly the point where the filing starts to matter.

So instead of describing my system, I packaged it. **The starter workspace is public on GitHub: the folders, the context templates, and five PM skills, personalized by one setup interview.** You don’t need to be technical to follow along; every step is spelled out, including installing the tools. And while my example is a product job, the same architecture fits any context-heavy work: a design lead’s research library, a data team’s metric definitions, a founder’s everything-folder.

Here’s the route.

- **The switch.**I’ll show you what chat mode and Cowork can’t give you, and why the terminal is worth it for a product manager.
- **The starter workspace.**Download it, run one setup skill, and give it a real task: six steps, each shown on screen.
- **The practices.**Six moves that make the system compound, each one naming the exact thing to do and what you get back.
- **The rhythm.**Here’s what a normal workday looks like inside the system, from the first short prompt to the two-minute filing habit.

The payoff is concrete. One sitting gets you running: install the tools, download the workspace, answer the setup interview, hand over a first real task. **You stop re-explaining your company every session, and every correction you make becomes permanent.** Every step is below.

Time to open the filing cabinet.

## What chat mode can’t give you

You probably run your product work through Claude’s chat today: a browser tab or the desktop app, maybe Cowork for the agentic tasks. It works, and that’s exactly the trap. Every conversation starts from zero. The strategy doc you uploaded last time is gone, the context you patiently explained lives in a thread you’ll never find again, and the output you liked is trapped inside a conversation instead of sitting in a file. **Chat is a great place to think and a terrible place to accumulate.** For occasional questions that’s fine. For daily product work, it means paying the same setup cost every single morning.

Claude Code is the same Claude with a different home: a folder on your machine that it can read, navigate, and write to. Context lives in files that persist. Recurring work lives in skills that run on command. Outputs land as documents you keep, not messages you scroll for. **In my daily use, everything I used to do in chat or Cowork runs better here, because nothing is ever explained twice.** That’s an observation from my own work, not a law, but it’s held for months.

The part that actually changed my job is the terminal underneath. **Claude Code speaks to GitHub natively, no connector needed: it can pull a company repository and answer questions about it, and it can push your own work.** Ask it to fetch the repo behind your product and explain what a feature flag actually does. Push your workspace to a private repository and a teammate can clone it and inherit your head start. For a product manager, that’s a direct line into the codebase and a way to share your whole system, both of which chat simply doesn’t have.

What nobody hands you is the filing system that makes all this compound. Anthropic’s best-practices guide is written, in its own words, “for engineers using Claude Code across various codebases, languages, and environments,” and engineers get their structure free: the codebase is already organized context. The PM guides that exist, Sachin Rekhi’s being the best known, rightly tell you to “store all of your product data and documentation in local markdown files,” then move on. Organized how? Try it without an answer and you get what I had: forty loose files and a stale setup. **The starter workspace is that answer, prebuilt.** Let’s set it up.

## Start with the starter workspace

The starter workspace is the structure I run my own product work on, cleaned into a template: the folders, the context files with guided placeholders, an example project, and five skills already installed. **The whole thing is free and public: claude-code-pm-starter.** Six steps take you from nothing to a first real task, and none of them assume a technical background.

One prerequisites note before you start. You need a paid Claude plan for Claude Code. **Everything else, including the two tools you’ll install, is free and covered in the steps.**

### Step 1: Install VS Code

Claude Code runs in a terminal, and a terminal alone is a bad home for a product manager: you can’t see your files. The fix is an IDE. **An IDE is just an editor that shows your folder as a panel of files while you work; developers live in one all day, and you’ll borrow it for the file panel alone.** Download VS Code, the free standard, and install it like any app. You won’t write code in it. You’ll watch your workspace organize itself in the left panel, which is exactly the visibility chat mode never gave you.

### Step 2: Download the workspace

Open the repository page, click the green `Code` button, and choose `Download ZIP`. No git, no account needed. Unzip it and move the folder wherever you keep your work. **This folder is about to become the place your product work lives**, so pick somewhere you’ll find it again, like a `work` folder in your home directory.

### Step 3: Open it in VS Code, terminal on the right

Open VS Code, choose `File > Open Folder`, and pick the unzipped folder. The left panel now shows the whole structure: `context/`, `projects/`, `operations/`, the skills. Then open the terminal inside VS Code with `Terminal > New Terminal`, and here’s the small trick that changes how the whole thing feels: right-click the terminal panel’s header and choose `Move Panel Right`. **Files on the left, agent on the right: it stops feeling like a command line and starts feeling like a colleague sitting next to your documents.**

If you don’t have Claude Code yet, install it now from the official guide; it’s one command pasted into that terminal. Then type `claude` and press Enter. **Claude Code starts inside the folder, reads the map file, and knows where everything lives from your first message.**

### Step 4: Say hello, then learn four commands

One distinction saves beginners a lot of confusion. **The terminal wants exact commands; Claude wants normal language.** Before you typed `claude`, you were talking to the terminal, where spelling and syntax matter. Now that Claude Code is running, you’re talking to Claude, exactly like in the chat app. So do the natural first thing: type hello and ask it what it can see in this folder. It will answer like a colleague who just read your files.

A few slash commands are worth knowing on day one, typed directly into Claude Code:

- **/model**- shows which Claude model you’re running and lets you switch.
- **/effort**- sets how much thinking Claude puts into each answer; higher effort for hard work, lower for quick drafts.
- **/mcp**- lists your MCPs, the live connections into tools like Jira, Slack, or Confluence, and lets you manage them.
- **/plugins**- opens the plugin system, and this one deserves a minute. Claude Code comes with Anthropic’s plugin directory available by default, and marketplaces can be added on top. One is built specifically for product managers: pm-skills, a hundred-plus PM skills across discovery, strategy, execution, and launch; its README has the up-to-date install instructions.- `pm-execution`covers PRDs, OKRs, roadmaps, and retrospectives;- `pm-product-discovery`adds interviews and assumption testing.- **Install the two or three that match your work, not all nine; every installed plugin is context the system carries.**

One more control, then you’re done with the tour: press `Shift+Tab` to cycle Claude Code’s three modes.

- Default mode asks your permission before touching any file. 
- Auto-accept applies edits without asking. 
- Plan mode is the one to remember: Claude researches and proposes a plan, and nothing changes until you approve it. 

**Start in default, use plan mode for anything big, and save auto-accept for work you’ve learned to trust.** For everything beyond these, this community cheat sheet is the best one-page reference I’ve found.

### Step 5: Run the setup skill

Type `/setup-workspace` and press Enter. The workspace ships with a skill whose only job is to personalize the template: it interviews you about your company, your product, your users, and how you like to work, then fills in the context files and renames the example project to your most active real one. Answer in plain language; it does the writing and shows you each file for a yes or a fix.

**You don’t have to answer from memory: paste links.** Hand it your product’s public site, a documentation page, or, if you connected the Atlassian MCP in the previous step, the Confluence page about your team, and it reads them and builds the context files from what’s already written. Fifteen minutes of this replaces the blank-page problem entirely. Your exact questions will differ a little from mine; each session is a little unpredictable, that’s just how these models work. Skip anything you don’t know yet; an honest placeholder beats invented content.

### Step 6: Give it a real task

Don’t end the first session on setup; hand it something you actually owe someone. Type `/status-update` and watch it draft your update from the project’s task file instead of your memory. Or point it at a real deliverable:

Draft the kickoff brief for [your project]. Match my writing preferences in

`context/preferences.md`. Check the open questions in the project’s`tasks.md`before concluding anything.

**The first output that sounds like you, about your product, with your project’s real state in it, is the moment the filing clicks.** Three more skills wait in the folder: `prd-review` for the next document you’d like a hard read on, `interview-synthesis` for your next user conversation, and `file-feedback`, the one that makes your corrections permanent.

That’s the start: a working system in one sitting. What makes it compound is how you feed it, and that’s six practices.

## Go further with six practices

The starter structure encodes the practices I converged on by getting each one wrong first. **They’re not a methodology to obey; they’re the moves that kept earning their place.** Each one below names the concrete thing to do and what you get back for it. The first one is the engine; the rest steer.

### Use it for every task, every day

The single highest-return habit is also the simplest to state: **default to the workspace for everything.** Each time a task lands on you, open the workspace and do it there with Claude Code, even when chat would feel faster. The first run of a new kind of task takes longer, I won’t pretend otherwise: **you explain, you correct, you adjust.** But every explanation lands in a file, so the second run starts ahead, and the third barely needs you. **It’s like teaching an intern: the first delegation costs you an afternoon, and then the work comes back done.** A task kept in chat mode teaches the system nothing; the same task done in the workspace is an investment that pays on every repeat.

The habit has a closing move, and it’s where the compounding actually happens. When a task is done and you’re satisfied with the result, don’t just close the terminal. **Ask Claude to bank what it learned:** convert the finished task into a skill if it will repeat, confirm the new facts into the context files, and update your preferences file with the feedback you gave along the way. I say it almost in those words: “we’re done, turn this into a skill, and file the corrections I made.” That one sentence at the end of a task is the difference between a tool you use and a system that learns.

### Sort files by how fast they change

The organizing question surprised me. It isn’t “what topic does this belong to?” It’s “how fast does this change?” Everything in your work has a speed: your product’s positioning barely moves, a status summary lasts about a week, a ticket queue changes by the hour. **File things by their speed, and stale can never masquerade as fresh.** File them by topic, and old truth sits next to current truth, looking identical.

**The three folders in the workspace are the three speeds.** The concrete habit: the next meeting note you take goes into `operations/` with the date in the filename, and the paragraph you keep retyping about your product goes into `context/product.md`. A note was true the day you wrote it, so the date travels in the filename where Claude reads it; the positioning paragraph is true until you change it, so it lives where every session can load it. Projects sit between the two speeds, which is why each project folder splits `tasks.md` from `brief.md`.

There’s a third speed, and you already own it.

MCPs, those live connections from Step 4, sit at the fast end of the spectrum: they’re for what changes hourly. Files are for what changes slowly. **If you’re pasting live data into a context file, an MCP should own it; if you’re re-fetching stable truth through an MCP every session, a file should own it.** If you’ve used PARA, Tiago Forte’s filing method, this rhymes with it, with one difference: PARA sorts by actionability, because a human’s failure mode is losing track of what to act on. **This system sorts by rate of change, because an LLM’s failure mode is stale context confidently reused.**

### Let Claude keep the map

`CLAUDE.md` is the one file every session loads: the map that says who you are, what lives where, and the few rules that always apply. Here’s the honest practice: **you almost never touch it by hand.** If you’re organizing an existing folder rather than using the starter workspace, the `/init` command creates it for you: Claude scans the folder and writes the map itself. From then on, maintenance happens as a side effect: when you add a skill or a context file, Claude updates the map to match, and your job is just to say yes.

The one thing worth knowing is why the map stays small. It loads into every session, relevant or not, so every line spends a little of what Anthropic’s context-engineering post calls the “attention budget”; their docs target under 200 lines and warn that bloated files make Claude ignore your actual instructions. **The map points, the files carry: knowledge lives in **`context/`**, loaded only when a task needs it.** I wrote about this just-in-time principle in How to build an AI helper for your team with Cowork; here, Claude applies it for you.

### Turn repeated work into skills

I carried a doubt about this practice for months. My habit is to look at every recurring piece of work and ask whether it can become a skill, so I give the feedback once and never redo the work again. It felt productive, and I wasn’t sure it was right, because nothing written for product people says where skills end and files begin. **The test that settled it: nouns go in files, verbs go in skills.** What Claude needs to know is a noun: product facts, user segments, terminology, in `context/`. What Claude needs to do is a verb: draft, review, synthesize, each one a skill.

In practice, skills come from the closing move of the first practice: **you finish a task, you’re satisfied, and you ask Claude to turn what you just did together into a skill.** The workspace’s `status-update` skill shows what one looks like: where it reads from, what shape the output takes, what quality bar it checks. Steps, a quality bar, an output format: that’s what a skill encodes, and that’s why “the roadmap review prep” or “the churn digest” qualify while “that one weird escalation” doesn’t. If you want to build one from scratch instead, How to build your first Claude Code skill walks through it step by step.

The test also catches the two ways things end up in the wrong drawer.

The first mismatch is a “skill” that’s really a fact bucket: product context wearing a skill’s clothes, never doing anything. That becomes a context file. The second is the reverse: a procedure written as notes inside `context/`, like a “how I run reviews” paragraph nobody can execute. **A documented-but-not-executable procedure gets re-explained anyway, which defeats the whole point.** That one becomes a skill. The know/do split is how documentation practice has organized itself for a decade (Diátaxis formalizes it); skills just make the “do” side executable.

### File every correction once

You know the moment. Claude drafts your status update, and you fix the same phrasing you’ve already fixed once. Here’s the reframe that changed my week-to-week: **a correction made in chat is perishable; it’s gone when the session ends.** The entire compounding return of the system comes from making corrections permanent, and this is where the practices stop being organization and start being learning.

The simplest version is the one to start with, and it’s the highest-return edit in this article. **Open **`context/preferences.md`** and write three lines about how you write: sentence length, what your documents open with, the words you ban.** From then on, every document Claude drafts comes out aligned with your writing preferences instead of sounding like a press release. That’s one correction, filed once, applied to everything that follows. The oldest entries in my own preferences file are corrections I have never had to give again.

For everything past that first edit, the workspace ships a skill that does the filing for you. When you catch yourself correcting something, run `/file-feedback`. It routes the correction with four questions.

Was it a one-off slip? Let it go; not everything deserves a file. A missing fact? It goes to the matching context file or the project’s brief. A wrong process? The fix goes into the skill, so it runs correctly every time after. Taste? It becomes a preferences entry, and here’s the detail that matters: **each entry gets a line on why and a line on how to apply it, so future sessions can judge edge cases instead of pattern-matching blindly.**

It turns out the vendor agrees, in almost the same words. Anthropic’s memory docs tell you to write things down when “Claude makes the same mistake a second time” or when “you type the same correction or clarification into chat that you typed last session.” The best-practices guide promises the payoff in one line: “The file compounds in value over time.” So here’s the version I’d put on a sticky note, and the line I’d hand your team: **if you’ve said it twice, it belongs in a file.**

### Ask before you assume

The last practice removes a blocker that sits in your head, not in your files. You’ll catch yourself assuming a thing is out of reach: too technical, too long, needs a tutorial, needs an engineer. **The move is to ask Claude Code first and let the answer surprise you.** I wanted to try AWS Amplify, Amazon’s service for building and hosting apps, and my reflex was the old one: follow the guidelines, read the documentation, budget hours I didn’t have. Instead I pasted the documentation link into Claude Code and asked for two things: a working prototype to test the service, and a step-by-step guide written for someone non-technical. **In one sitting I had something I could actually release, plus a guide I could follow and understand.** The hours I had budgeted were never spent.

The same reflex applies at every scale, not just new tools. Before you summarize the forty-page research report yourself, ask. Before you assume the data question needs an analyst, ask. Before you check every intermediate step, give the whole task with the constraints and verify the result instead. **The system you’ve built through the first five practices is exactly what makes asking cheap: **the context is filed, the preferences are known, and a wrong answer costs you a correction, which the system then keeps. Six practices, one architecture. Here’s what it feels like on a normal workday.

## Run your day through the workspace

A day inside the system looks less like chatting and more like working with a very fast colleague who reads the files first.

Sessions start in the project folder, not a blank chat. I open the workspace in VS Code, launch `claude` in the terminal, and the map loads with every file path ready to use. The task file already says where things stand, what’s blocked, and what’s next, because the previous session updated it. My first prompt names the goal and points at files by name, and it stays short on purpose. **Prompting shrinks as filing grows.** A good prompt in this system has three parts: the goal in one sentence, the non-negotiable constraints, and what to verify before showing you. About a hundred words, often less. The steps live in skills, the background lives in files, and the prompt only has to steer. For bigger work I add one move: switch to plan mode with `Shift+Tab` and approve the plan before anything runs.

The habit that holds it all together takes two minutes at the end of a session, and the `file-feedback` skill carries most of it. Three questions: did I correct anything I’ve corrected before, did a new fact surface that isn’t filed anywhere, did a piece of work repeat again? **Repeat corrections go through the router; repeated work gets nominated as the next skill.** Skip the ceremony when the answer is three nos, which is most sessions.

Every dozen sessions or so, I spend fifteen minutes pruning, and I’d argue this is the least optional part. **Stale context is worse than missing context.** An expired competitor note or an outdated positioning line doesn’t look stale to Claude; it gets woven into new work with full confidence, and Anthropic’s best-practices guide carries a matching warning: when two rules contradict each other, Claude may pick one arbitrarily. So: date-stamp what’s dated, archive closed projects, delete anything you wouldn’t want quoted back to you in a draft. That’s the whole system, running. What’s left is to start it.

## Let the workspace earn the rest

What changed for me isn’t that Claude got smarter. It’s that I stopped having the same conversation twice. The company explanation I used to retype lives in `context/`. The feedback I used to repeat lives in skills and a preferences file. The projects pick up where they left off. **The system’s real product is quiet:** shorter prompts, fewer corrections, sessions that start in the middle of the work instead of at the beginning of my job description.

The shape travels well beyond product management. A discovery-heavy PM files interview syntheses as dated outputs and lets a skill do the synthesizing. A platform PM’s stable context is API consumers and versioning policy. A founder turns the investor update into a skill fed by a metrics file. A design lead files critique standards once instead of re-explaining them in every review. A data lead’s metric definitions become the context file every teammate’s sessions read. **Different jobs, same three drawers, same habit.**

Start smaller than feels ambitious. Download the starter workspace, run the setup interview, and add those three lines about how you write. Then bring it your very next task, and the one after that. **The next time you catch yourself telling Claude Code something for the second time, you’ll know exactly where it goes.** And soon enough, you won’t remember the last time you said it.
