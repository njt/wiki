---
url: https://lethain.com/software-factory-experiment/
date_fetched: 2026-09-25
---

# Trying the Software factory pattern.

One of the interesting challenges of the AI ecosystem in 2026 is that new, effective patterns emerge faster than I can adopt them. I’ll find a handful, get back to work, and realize a month later that I’d missed four or five more. The adoption cycle for Imprint this year has been something like:

- January: get every engineer onto Claude Code every single day
- March: ok, let’s also get everyone else onto Claude Code or Claude Cowork every single day
- April: local development is bottlenecked on checkout and worktree model, instead create ~10 local workspaces which each have an independent checkout of every repository, and operate at the workspace level, not at the repository level, so it can generate cross-repository pull requests across frontend, backend, infrastructure and data monorepos
- June: oh boy, agent-driven development is heavily constrained by lack of a common task management system with higher visibility and less permission complexity than Jira, so let’s migrate the entire company over to Linear and hard stop on Jira
- July: yikes, now we have visibility into all these tickets, many of them are trivial but managing them through local development isn’t scaling, let’s roll out an orchestrated harness which internally we call “Agent Fleet”, along the lines of Stripe’s Minions

The most recent question for me has been figuring out how to adopt the software factory pattern. (After some light research, the specific AI-context origin of this term is slightly messy to attribute, but I think it might be Justin McCarthy in February 2026’s Software Factories And The Agentic Moment.)

The software factory pattern is looping on a broad goal, and then relying on the harness to drive progress towards that goal. Our first pass at implementation is fairly basic:

- An agent skill - `/linear-project-loop`which reads in a Linear project and starts by auditing that project’s goal definition on these dimensions:- An RFC in Notion that describes the project’s goals, how those goals are measured, and the general approach
- A Datadog dashboard or Snowflake queries that measure progress against those goals
 - If those are missing, or the Linear project is missing in its entirety, it iterates with you on creating those missing tools. 
- Then it reviews the state of the metrics and issues for the project. If new work is identified, it adds those issues to the project. It updates the state of issues that have moved. 
- It works on the non-blocked tasks based on the project’s current state. This is often writing a pull request, updating a pull request, pinging for review, asking a clarifying question, etc. 
- When a task completes, if the project description is fresh, it takes on the next task. If the description hasn’t been updated in a while, it reruns the loop starting with the first step. 

Right now I am running this locally in a local harness, but it’s working well enough that I anticipate moving the behavior to be driven by the same orchestrated harness that we assign one-off tasks to.

What I particularly like about the factory pattern is that it parallels very closely how I’ve been working locally, while forcing me to recognize the places where I was accidentally hording parts of the state for myself regarding the goals of the project. I was already asking agents to iterate on specific Linear projects, but they didn’t have the ability to evaluate if they were going in the right direction, or if it was missing necessary tasks. Now it does. The other place this has been extremely helpful for me is checking in on projects post release. For example, I shipped our passkeys implementation earlier this year, but some months go by without my checking in on how it’s going. If we saw adoption spike, or error rates start to turn, I might miss it, but running the factory in a less frequent post-release mode would catch it immediately.

The final thought that’s been interesting to me is how much all of the pieces here compound only to the extent that you have the other pieces. For example, this factory pattern depends on having Datadog MCP and Snowflake access available to manage goal-tracking, but it also depends on Linear being the single source of state for the company’s work, and an orchestrated harness that can perform work independently from your laptop. Keeping up with this many migrations is a fascinating industry moment.
