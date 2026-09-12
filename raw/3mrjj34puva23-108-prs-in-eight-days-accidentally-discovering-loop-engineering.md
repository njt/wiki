---
url: https://brittany-ellich.offprint.app/a/3mrjj34puva23-108-prs-in-eight-days-accidentally-discovering-loop-engineering
date_fetched: 2026-09-13
---

# 108 PRs in eight days: Accidentally discovering loop engineering

For the last week or so I've had an agent working my task board. It picks up a task, writes the code in an isolated worktree, opens a PR, gets CI green, responds to review feedback, and parks the result somewhere I can test it. Then it sleeps for a while and does it again. As of this morning, 108 tasks have gone through it in about eight days. Compared to my previous average of 5-10 PRs per week, that's a pretty substantial change.

There are a few things that enable this to work well. I am on a team that is small and is cooking right now. I'm not the only person on my team moving fast, and we have a very high level of trust. We don't have a mandatory rule that another person looks at the code, too, which helps tremendously. I still get input from a coworker if I'm unsure about something, but not having to wait on someone else's timing to merge and get stuff done is huge.

__I posted about this on Bluesky recently__ ("unless you're shipping nuclear safety code, you probably don't need a human reviewer for your PRs") and the most consistent pushback wasn't about AI review quality at all. It was that PR review is how teams stay aligned and how junior engineers learn. I think that's fair, and it's exactly why the team context matters here: on a two-to-three person team that talks every day, alignment happens in conversation, not in a review tab. If you're on a bigger team, or mentoring juniors, that might not work as well. I also want to be clear that I didn't remove the human from review. I moved where the review happens. I still look at every change, but at the testing-and-outcome level instead of line by line in a PR.

We also don't have continuous deployment (CD) set up. That's a weird thing for me to say as a positive, because for a very long time that was a hallmark of good software engineering practices... but with me shipping 108 PRs in a week, I actually have been thinking differently. Now I prefer CI plus a preview/stage environment, releasing a batch of work at once instead of shipping a new build to users on every merge. It still averages 2 or so changes in production per day, but it dramatically limits the surface area for when things can go wrong. It means I only have to regression test once or twice per day, and can handle bugs early on, before they're in front of users.

It turns out that what I've been iterating on for my personal process has a name: __loop engineering__. Addy Osmani coined the term in June, building on things Boris Cherny (Claude Code) and Peter Steinberger had been saying about designing the system that prompts your agents instead of typing every prompt yourself. Most of what I've read about it stops at "give it a clear goal and a turn cap." That advice is fine, but I found for my own use case I need more than just that.

**My three layers to loop engineering**

The system has a protocol, a loop, and a worker.

The protocol is a markdown file with the rules: what each status means, what happens in what order on each pass, what gets merged and what doesn't. It lives next to the board rather than in the repo, so editing the rules and editing the board are the same act. I keep my board separate from the main repos I work in, so that I can work in multiple repos from a single source.

The loop reads the board, runs gh, updates task frontmatter, and dispatches work. It writes no code. It doesn't touch the repository working tree at all.

The worker writes code in its own isolated worktree and reports back with a small structured block. It never touches the board, and it never talks to me directly.

**The board is a folder filled with markdown task files**

My first version was a single markdown file with a table in it. That lasted about a day, as I realized it caused a lot of collisions between me adding to the board and the board updating itself.

One markdown note per task fixes this structurally, and works really nicely with __Obsidian's Bases__, which provides the actual board structure, with a bit of frontmatter to keep things tidy. We only collide when we both touch the same task, which is rare and obvious when it happens. As a bonus, a stray | in my own notes can't break the whole system anymore, and the board becomes queryable as data instead of something that has to be parsed as text.

The same reasoning applies one layer down: the loop is the only writer to the board and to the memory file. Workers hand results back and the loop records them. Three agents running concurrently can't corrupt a file that only one process is allowed to write to.

**The main checkout is mine**

Every task runs in a subagent with worktree isolation, branched from the repo's default branch, not from whatever I happen to have checked out. My half-finished local state never leaks into a task, and a task's edits never show up in my tree while I'm in the middle of testing something.

I test tasks that are ready to test in my locally checked out branch. From there I can review the code, make changes myself, or add changes to the task for the agent to take and change on the next loop.

**The loop has to be able to stop**

I run my loops with Claude Code's /loop functionality with no interval.

Giving /loop an interval turns it into a cron job: it runs until you kill it or it expires, and it can't end itself. Bare /loop without an interval is self-paced. It picks its own delay after each pass, short while a PR is active and long once things go quiet, and it can end the loop entirely once the board says nothing can move without me. This lets me set up a bunch of issues for it to churn through and run through them over night, creating a nice backlog of things for me to test with test instructions included for the next day.

The protocol has an explicit stop condition, written as a list of things that must all be true: nothing in progress, nothing waiting on CI, nothing merge-ready, and every remaining task either waiting on me or finished. When that holds, the loop says what it's waiting on in one line and stops until I start it again, meaning that I finish using tokens as soon as it runs out of work.

**Memory, with a hard cap**

The loop keeps a memory file of what the process has learned: facts about the codebase that cost a worker real time, preferences I've stated more than once, defects that keep coming back. Each entry carries a confirmation count and a date, and every dispatch pastes the memory into the worker's prompt. It forces the unwritten stuff to get written down, with a count attached, so we know how often a problem is run into and resolve it, instead of continuing to get stuck.

Once the confirmation count on a bug or issue reaches a certain point, fixing it becomes its own task. For example, workers kept getting stuck on a stale skill that was saved in the repo, until they eventually made a task to remove the skill. That makes the entire process more efficient over time, and frees up a memory slot for something else.

Using a hard cap keeps the memory list streamlined and requires that the system make tradeoffs to find the most relevant pieces to keep. Memories get loaded into each worker as they work, and I want to manage that context to keep it small.

**It’s me, hi, I'm the bottleneck, it’s me**

My board right now has zero tasks queued and eight waiting for me to test them, which means it's waiting on me to do the testing and to add more work to the queue. Where I'm spending my time gives me good insight on how to make the loop even better.

Several people in that Bluesky thread predicted this shift: that human effort would move from reviewing code to reviewing specs and issue definitions. That matches exactly where my time went. Both ends of my pipeline are spec work now: deciding what goes in the queue, and verifying what comes out. It almost feels like being a software engineer in 2026 is just being a product manager and QA.

For example, I ended up adding a skill to create a task so that I can explore an issue with Claude, like perhaps some errors in our error log or something operating slowly, so that Claude can quickly add tasks to the backlog as we explore. Trying to reduce the bottleneck of adding the right items to the queue is the next area I’m iterating on.

Automating the testing and verification stage is the other area I'm looking into. How can I create an automation that will do the manual review process before even handing it over, to check things like accessibility, how things look on a mobile device, or whether there are any regressions? That way my QA stage continues to get shorter.

**How to get started**

I'm open sourcing __my loop setup__ for you to try out if you want. Everyone's process is different, so you may even want to start from scratch.

As a first step, I recommend writing down what "done" looks like before you write anything else. Every good decision in my protocol came from being forced to define a state precisely: the task completed, green CI, and an approved AI code review. From there you can start slowly working out from "done". On the frontend, how you go from a task or issue to a completed PR, and on the backend how you make sure everything gets prepared for you to review.

If you give this a try I’d love to see how it works for you! Reach out and let me know.
