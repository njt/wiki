---
url: https://www.usecontextlayer.com/log/a-platform-for-bespoke-software
date_fetched: 2026-11-09
---

I think we need a new kind of platform.

I want to vibe-code a bunch of tools that are quick to make and simple to change, that are connected to my data and my life, that I can share with others, that look good out-of-the-box, and that I know are secure. Turns out, all the pieces exist, but there's still an awful lot of plumbing and wiring to do. And it's pretty unapproachable for non-engineers and not all that optimized for agents.

I've been shaping and iterating on this idea of slates to solve this. I'd like to share the design decisions and observations that underpin them.

Two things have changed in the last year:

- AI has become spectacular at writing code that works.
- Folks who aren't engineers are using AI to write code to accomplish things.

These changes make the first design decision simple: **it has to be agent-first**.  That doesn't just mean agents manage things for you; it means agents are the only ones writing code. We're really building this for anyone *who wants to use agents* to shape their tools.

On a side note - software that is supremely engineered is not always the same as software that is supremely valuable. I like to build software that is supremely engineered. I strive for ContextLayer to be supremely engineered. The software that people build on ContextLayer should be first and foremost supremely valuable.

The next decision falls out of the first: **we can be super opinionated about the stack.** If agents are the only ones that are writing code, then it doesn't matter what language or framework the code is written in. LLMs don't care. So we removed the choices altogether. This has incredible benefits. We get to focus on packaging and composability. We get to choose existing libraries we love and wire them together. The stack is TypeScript, React, React Router, Tailwind CSS, Kysely, PiDurable, and gigatons of Cloudflare. We haven't bothered with anything else. The result has become a genuinely fun and simple way to build things. I get to focus on just the problem I'm trying to solve, and none of the other drudgery.

Then we thought about security; not in some abstract way, or in some bureaucratic ISO-standard kind of way, but in a relatively plain way. To put it simply, *I feel more comfortable with vibe-coding when I know that the blast radius is small.* There shouldn't be a million ways to move data in and out of a slate, or even two ways for an agent inside the slate to effect change outside the slate. There should be one way. If the agent really is going to delete all of my files, there should be only one way for it to do so, because if there's only one way, I can observe this way. I can monitor it, constrain it, and keep it under control.

I think MCP is the right answer for this. All data and effects in slates are powered by MCPs. There's no other way for data IO.

**Code inside the slate is considered to be untrusted.** I want to treat it like a black box. I don't want to have to review code. So we run slates on Cloudflare's Workers for Platforms, and all egress except via authorized MCPs is blocked. Even the code that runs in the browser is not allowed to access the broader internet. We've locked it down with Content Security Policy headers.

It's really just about minimizing the blast radius of vibe-coded code.

Did I mention that we love simple and composable?

- In its simplest form, a `slate`is just a folder.
- `git`repositories are baked into the platform. There's no need to set up GitHub or anything else.
- Deploying code is as simple as `git push`. We handle database migrations too.
- Your agent can manage everything with the CLI (or use the dashboard)

Third-party data access is a big one. It makes zero sense for every single slate to build its own authentication and authorization system. So we baked that into the platform too. And we wired up Composio so you get access to 1,400+ integrations out-of-the-box (which means I don't have to spend time and tokens on them).

Then we tried something wacky: we don't bother telling the slate who the logged-in user is. Instead, resources (database, folders, connections to third-party services) are **injected into the slate at runtime**. The code inside the slate doesn't know and doesn't care who these resources belong to. The platform provisions them and binds them to the slate. When you create a slate and declare what resources you need, you also declare whether a resource belongs to the viewer or the owner. If I declare an email connection as belonging to the viewer, we can both use the same slate: it accesses my inbox when I open it and yours when you do.

And of course there are agents. There are scheduled agents. There are agent jobs. And there are good old-fashioned agent chats.

We're experimenting with some other really fun things:

- We can expose the loaders and actions in the React Router app as MCP tools, no wiring needed
- We can implement forking where agents help keep your fork up-to-date with upstream. Actually, we think forking is the primary way people will use other people's slates. It's going to be the default.
- We're making it so that your agent (Claude, Codex, Hermes, etc.) can connect to a slate with MCP, and then access the live React state of every browser session you have open for that slate. It means that your agents can do stuff alongside you in the slate.

It's a fascinating time to build software. We get to rethink foundational assumptions about how software is built. We’re putting identity, data, apps, and agents into the same environment, and making that environment something you can shape around your own work. Give us a little bit of time and we're well on our way to rethinking operating systems entirely.

Have fun! We're excited to see what you make.
