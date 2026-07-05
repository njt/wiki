---
url: https://wingedpig.com/2026/02/26/if-ai-is-doing-the-investigation-version-the-investigation/
date_fetched: 2026-07-05
backfilled: true
---

# If AI Is Doing the Investigation, Version the Investigation

By **Mark Fletcher**

I was looking at a bug report about calendar feed generation in Groups.io. ICS is one of those specs where interoperability with other calendar systems is a constant source of complexity - the spec is opaque and every client interprets it a little differently. I’d fixed a similar bug a month earlier, so I went back to the commit. The diff was there, but the Claude Code session that had helped me figure out the problem was gone. I was trying to reason through it with only half my brain.

This keeps happening. I’ve been using Claude Code for close to a year now working on Groups.io; it’s been incredibly helpful to me. But there’s a bunch of knowledge tied up in those sessions that isn’t getting preserved. I commit bug fixes, but all the knowledge around finding and fixing the bugs is lost. I commit new features, and the thought processes that went into developing them is gone. Revisiting a commit a month later means staring at a diff with no context for why those changes were made.

I’ve spent the last several months re-working my development environment. The result is Trellis, my open source development environment. It makes it easy to switch worktrees, view production logs, run distributed traces, and spin up Claude Code sessions. Crucially, it makes it easy for Claude to safely run investigations of bugs in production. As part of this work, I wanted to address this issue of not preserving Claude Code sessions with commits, which is where Cases come in.

A Case is a directory. It’s got a `notes.md` file with a human summary, a Claude transcript, a set of distributed trace results - everything involved in researching and fixing a bug or developing a new feature. It gets committed alongside the code changes.

```
cases/2026-02-24__fix-mybucket-get/
  case.json
  notes.md
  transcripts/claude__session-3f8c.json
  traces/web.app01-g2.130812...json
```
Most Claude sessions are throwaway, same as most terminal sessions. Cases are for the ones worth keeping. You don’t write a postmortem for every alert; you write one when it matters.

It’s not just bugs. When I’m building a new feature, the Claude session often has the design reasoning — why I went with approach A instead of B, what constraints I was working around, what I tried that didn’t work. That’s the kind of context that’s invisible in a diff but matters six months later when someone (probably me) needs to change it.

Because they’re in the repository, they’re available to all developers. They show up in PRs alongside the diff, so reviewers can see not just what changed but why. They show up in `git log`, so when you’re trying to understand a commit six months later, the investigation is right there. And they travel with the branch - no external wiki to keep in sync, no links to go stale.

Trellis can also read them and re-hydrate Claude Code sessions for continued work later. If that had existed when I hit the ICS issue again, I could have picked up where I left off instead of starting over.

One design choice worth mentioning: Claude in Trellis has read-only access to production systems. It can query logs, run distributed traces, and read crash reports, but it can’t restart services or modify anything. That’s deliberate. Investigation tools should be safe to run freely. And once you’re committing transcripts as permanent records, it matters that the AI wasn’t doing anything you’d need to audit for side effects. The transcript *is* the audit trail.

I built this into my development environment, Trellis. But the idea isn’t Trellis-specific.

If AI is part of how you build software, its work shouldn’t vanish when you close the tab.

If the investigation is worth doing, the record of it is worth keeping. And if it’s worth keeping, it should live where the rest of your work lives.

If you want to see this in action, here’s a short demo.
