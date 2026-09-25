---
url: https://www.geocod.io/code-and-coordinates/2026-09-23-the-year-of-internal-tools
date_fetched: 2026-09-25
---

# Code and Coordinates

engineering at Geocodio

# The year of internal tools

At Geocodio, it has always been important to us to automate processes and build our own tools and helpers, so we can save time and be more efficient. For almost ten years, Geocodio was just the two of us. Automation is a big part of what made that possible.

Those tools have saved us countless hours over the years. Local bash scripts that set up a development environment or download the specific data files we need for geocoding. Infrastructure scripts that check the health of the load balancer or sync some data down. Small scripts that make the on-premises release process faster and more efficient, with fewer manual steps and less room for human error.

One of the bigger ones came along well before the AI era. We built our own deployment tool. It visualizes the deployment process, so we can easily see how far along a deploy is and what is live right now. If something goes wrong we can see it and roll back. It handles deployment freezes and a bunch of other things that would be a lot more cumbersome and a lot less transparent as a manual process.

All these small scripts and helpers and various tooling have been helpful over the years and have had a positive impact. But it pales in comparison to where we are at in 2026 and what we are able to do now.

## What changed

This is the year of internal tools. We have gone from creating small bash scripts here and there, artisan commands in Laravel, small isolated repos with code to generate reports, to being able to really raise our ambitions and create full-fledged internal apps. Mobile friendly, good UX, fantastic test coverage.

What allowed us to do that is the advent of AI and frontier models that let us build at a much higher level than we have ever been able to before.

I must admit that in the beginning I was hesitant to start taking on these larger projects. The thing that always got in the way of the bigger, more ambitious ideas was not the building. Even pre-AI you might be able to knock out some kind of awesome internal tool over a couple of days. But then you have the maintenance burden. You cannot keep building tools if you do not think about the fact that you have to maintain them, keep them working, keep them up to date, fix the bugs you find.

That is the other side of the coin, and it is the side AI has also allowed us to solve. I am not worried about building all these internal tools and maintaining them, because I can use AI to keep up with the maintenance as well.

## We have a system for it now

We have gotten to the point where we have specific systems in place that let us quickly scaffold and create a new internal tool. We recently launched our console-ui package, which includes Tailwind CSS tokens as well as shared React components, so all these internal apps share one component library instead of each one growing its own.

We have also built up enough experience with AI that we have some solid principles for how to give these tools a strong foundation, so they are built for the long term:

- A significant amount of planning before we take on the job of building anything. 
- A lot of research, including engineering spikes that prove out a concept first. 
- Numerous Grill Me sessions, using Matt Pocock's wonderful Grill Me skill, which interrogates a plan until the weak parts fall out. 
- Iterating on UI mockups, which answers a surprising number of questions and resolves problems before we go and build the real thing. 
- Using powerful models like Fable in the planning process, to make sure what we are building solves the problem we are trying to solve, in a sustainable and well-engineered way. 

On top of that, we have also built up high quality internal standards for these projects. Testing, CI, static code analysis, best practices for authentication, that kind of thing.

That is also where the time goes. The planning and the spec take weeks. The implementation takes hours or days. Some of these apps we had been thinking about for years before that, so the spec was mostly a matter of collecting what was already in notes and meetings and turning it into a project. Atlas is the clearest example. It had been in my head for years, and once the spec existed, building it was the fast part.

## Every tool ships with its own documentation

Each of these tools has a full documentation subsite. A real guide on how to use the tool and how each feature works, with screenshots and examples.

A lot of it is AI-generated, at least the initial draft. Even so, having that consistency across every tool is really nice. It communicates what a tool does and how to use it much better than a README ever did.

It also solves a problem I did not expect. When you have been sitting there writing big specs and plans for weeks, you sometimes lose touch with what you built. Writing the documentation brings that back into focus.

Every one of these repos has a rule in its CLAUDE.md saying that any change affecting the documentation requires Claude to go and update, add, or remove the relevant parts. It still needs a human eye here and there, but it is a solid base rule that keeps things current. Documentation that is not up to date is useless.

This is the rule from the Atlas repo, lightly trimmed:

```
### Keep both sections in sync with the app
Whenever you make a material change you MUST update the documentation in the same change.
A user-visible change lands in the user guide; a change to architecture, dev
workflow/tooling, infrastructure, deployment, integrations, a pipeline/job/schedule, a
scheduled command, a deploy/secrets change, or an Environment API contract bump lands in
the internal docs. Many changes touch both.
- **Changed behavior** → edit the relevant page(s) so they describe how the thing actually
  works now. Don't leave stale screenshot markers, steps, class names, paths, commands, or
  claims.
- **New feature or new maintainer-facing surface** → add a new `
```
.md` in that
  section's directory and register its slug and title in the matching `DocsManifest`
  constant. The manifest is the single source of truth for which pages exist, their order,
  and their sidebar grouping.
- **Removed capability** → delete the page **and** its manifest entry, and any cross-links
  to it from other pages.
- **Renamed/moved** → update the slug in both the file name and the manifest, and fix
  inbound links. ## What we have built

We are taking on bigger challenges now, like building our own customer support platform.

- Atlas for customer support. It is built, and we are rolling it out now. 
- Bullpen for managing, organizing, and planning our sprints. 
- Yak, a coding agent for papercuts. It picks up small tasks from Slack, Linear, Sentry, and GitHub and comes back with a pull request to review. Yak is open source, so anyone can download it and run it against their own repos. 
- Our deployment tool. 
- A compliance tool that covers the parts our compliance platform cannot do, and then pushes the results back into it. 
- Treehouse, which gives every git branch its own worktree and its own docker-compose stack, so several branches or agent sessions can run at once without colliding on ports. Also open source. 

Our infrastructure engineer is also working on a tool for managing infrastructure, handling CVEs in our dependencies, and making regular maintenance easier with CD jobs on top of GitHub Actions. That one is a good illustration of the maintenance point above. Some of the tools exist to keep the other tools healthy.

## Why the support platform is worth building ourselves

The big benefit of building our own is that we can make it extremely well integrated with our workflows and our data.

Right now, helping one customer can mean going into three or four different tools to get the information you need. Our current support tool. An admin tool, and we have two of those, one for self-service and one for enterprise. The billing tool. Analytics. Having both self-service and enterprise customers makes the picture more complex for us than it would be for most companies.

The whole point of building the support platform ourselves is to bring all of that into one place. The person replying to a customer can see exactly what is going on with that customer, what kind of account they have, and what has happened recently, in a UI that is streamlined for the exact workflow we have at Geocodio. They get the information and the actions they need at their fingertips, which means we can solve the problem much faster.

## What we are not going to build

None of this means we are going to replace every SaaS product we use with our own tool, or that we are going to go build dozens of new internal apps. There is a limit, and there is a fine line between what should be in-house and what should be off the shelf.

Email stays with Bento, and we are happy with it. Running our own email infrastructure is a responsibility that makes no sense for us to take on. The risk is too high, and it is not something we have any experience with. Twilio keeps the communications work. Nobody here is writing a Slack. There is plenty of good tooling out there.

Three questions sort it for us:

- Do we have real experience in this domain, or would we be learning it on the job? 
- What happens to the business if this is down for a day? 
- Does the value come from being integrated with our own data and workflow, or would an off-the-shelf tool do the job just as well? 

Just because tools are easier to maintain now does not mean we can take on fifty of them. We have to be deliberate and thoughtful about the way we build them. This is also why we build them on shared infrastructure, shared UI, and shared principles, so they are easier to maintain and share as much code as possible. We are not trying to create a micro-internal-tool-services architecture either. We want tools that cover a whole feature set, rather than inventing a new service every time we have an idea.

## Every tool you build is a responsibility

That trade-off is real each time, and the part that is easy to forget is uptime. Internal tools need to be reliable too. What would happen to our business if we could not reply to customer support requests for a day because we shipped a change and broke something?

We got a small taste of this recently. We have an internal ETL tool for importing and working with address data. We did a massive upgrade of it, and then it was unavailable for a day while we ironed out a few issues with the deployment. In this case it did not affect keeping the lights on. If the same thing had happened to Atlas on a busy day, it would have been a different story.

That day was the result of a trade-off we made on purpose. These tools do not have a staging environment. A staging environment for every internal tool is one more thing to keep running and keep in sync, and for tools that only we use, we decided that cost was not worth paying. Changes go to production, and local QA, test coverage and CI carry the weight that staging would otherwise carry. The ETL outage is what that decision looks like when it goes wrong, and we accepted that when we made it.

The other half of the responsibility is access. Atlas can see customer accounts and billing, so who can reach it matters as much as whether it is up. One big benefit of hosting these tools ourselves is that they live entirely inside our private network. They are not reachable from the public internet at all. You have to be on our VPN before the sign-in page even loads, and only then do the usual authentication rules apply.

## Where this leaves us

I do not know what the future will bring. We have to be careful not to end up maintaining tons of internal tools, or creating tools for the sake of creating tools.

But if these tools help us do our work better, make us more efficient, and reduce the chance of human error from manual processes, then we can build a better product, we can give our customers a better experience, and we can improve the quality of life for our team.

If you are thinking about this at your own company, start small. Find the process where you already work around the gap between two tools by hand every week, and build the thing that closes it. That is where we got the most value the fastest, and it does not require you to believe that anyone should rebuild a SaaS product from scratch.

### Get new posts in your inbox

We write about what we're working on, thinking about, and getting so excited playing around with that we accidentally stay up a bit too late.
