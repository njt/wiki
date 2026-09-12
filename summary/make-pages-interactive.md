---
url: https://xcancel.com/paraschopra/status/2059167147516199152
title: "Paras Chopra on interactive HTML iteration with Claude Code"
author: Paras Chopra (@paraschopra)
date_fetched: 2026-07-05
date_published: 2026-05-26
topics:
  - claude-code
  - agent-coding-workflow
---

Thread by Paras Chopra introducing his Claude Code skill "make-pages-interactive" which enables generating static HTML files as outputs (reports, explorations, code structure, mockups) and iterating on them via browser comments that Claude Code watches and responds to. Published May 26, 2026.

## Full Thread Content

Paras Chopra (@paraschopra):

My favorite way of interacting with Claude Code is to have it generate static HTML files as outputs (reports, explorations, code structure, mockups etc.)

I wanted to iterate on the file by commenting in browser and having Claude update the output live.

So, I built this Claude Skill

How it works:
- Install Claude Code skill (ask it to clone repo)
- Build an HTML page for anything (e.g. research coding agents and generate HTML report)
- Ask it to make the page interactive

That's it. CC will launch a localhost server and allow you to then leave comments on the page itself and once it updates, will give you a tour of changes.

It's like Google Docs kind of comments/iteration but for HTML pages.

Give it a try: github.com/paraschopra/make-pages-interactive

## Notable Replies

**Aggy | Abhishek Agarwal (@AggyAbhishek)**: This is natively built in on codex, by the way, and I've been using it for that, and it's pretty incredible. They have an in-app browser which has a pretty neat annotation as a default mechanism of interacting with any browser page.

**Paras Chopra**: ah, didn't know

**Utkarsh Sengar (@utsengar)**: And ask CC to host it here htmlbin.dev

**Paras Chopra**: gist pages seems better htmlpreview.github.io/?gist...

**Nate Voss (@natevoss_dev)**: You're basically showing Claude instead of telling Claude. That gap is probably bigger than it seems once you're iterating at speed.

**Paras Chopra**: yeah, this is 100% true

**Chandra Vattikuti (@VCShekhar)**: Sweet timing. Assuming this is good only for the author prompting Claude and would need some sort of a hosted solution if the team had to make joint edits / post comments i guess.

**Paras Chopra**: yeah, this is for self-iteration. but you could push the folder and it contains feedback as well

**Abhilasha Purwar (@AbhilashaPurwar)**: Holy moly, literally been trying to make exactly this. My entire workflow across everything - ppt, doc, proposal, review doc, product iteration, startups slides, newsletters, blogs, has become html. Tbh, between .md, .html, vs .pptx/.docx etc - .html has so many benefits at 50% or less token as .pptx or .docx; Will give this a try. Quick question: do you think this could also be a CLI / npm-style tool? Something like: npx make-pages-interactive ./report where it injects the commenting layer, launches localhost, stores comments in a structured inbox, and then any agent — Claude Code, Codex, Cursor, etc. — can read the feedback and patch the HTML. So maybe the skill is the Claude-specific interface, but the core primitive is agent-agnostic: "Google Docs comments, but for local HTML artifacts."

**Paras Chopra**: yeah, just ask claude code to spin it into cli. Should be pretty simple

**Tarun Firodiya (@tarun_firodiya)**: Similar thing for .md if you want to save those tokens

**Nathan Baschez (@nbaschez)**: Introducing Roughdraft! A new open source project designed to make collaboration with agents better. The idea is to bring commenting and suggested changes to markdown (e.g. plan docs) in a nice interface. Free, local, etc. roughdraft.md

**Paras Chopra**: yeah this is similar but HTML gives a lot more interactivity!

**Bob Ulrich (@optic0n)**: hah. this is awesome. a colleague of mine and i built something similar in the last week! this is much more polished. where do you store the comments? can they persist when the webserver dies? we're trying to create a lightweight google docs comment style flow for commenting on technical docs

**Paras Chopra**: there's stored locally in the same folder as html file. they persist

**Aarthi Ramamurthy (@aarthir)**: very cool and super useful for charts/graphs that I generate (usually dump them as html files so I can browse through).

**Dean Sacoransky (@deansacoransky)**: This should be native to cc

**MagicPath (@MagicPathAI)**: This is great! But instead of having a bunch of local HTML scattered around, you can ask Claude code to build these things in MagicPath.

**Matt (@m13v_)**: the iteration loop works until claude code auto-compacts mid-session and silently drops the context your in-browser comments referenced. fazm wraps the same loop with the compacting disabled so the full session stays live

**Victor (@victor_explore)**: html files are the new whiteboard, except this one writes itself

**Nav Toor (@heynavtoor)**: google docs for HTML

**BowTied SaaS Guy (@BowTiedDolphin)**: Isn't this what Claude design does

**Adel Bucetta (@adelbucetta)**: that's cute, but the real challenge is making claude's updates trigger a version control system so you can see history and roll back changes

**Vatsal (@vtslkshk)**: This is now native in Claude Code desktop app. Open the HTML in preview mode and you'll find the element selector on top

**Mustafa Ergisi (@mustafaergisi)**: This is exactly the loop I fell into for my pipeline dashboards. Static HTML out of Claude Code beats hooking up a real frontend every single time. Even my agent run summaries live there now.
