---
url: https://lovable.dev/blog/how-we-built-lovable-and-why-we-keep-rebuilding-it
date_fetched: 2026-11-09
---

Making Lovable simple and dependable has meant repeatedly rethinking how it works. As the technical landscape keeps changing, engineering choices that once helped can start holding you back. Many teams building AI-powered products face the same question: what should you keep, and what should you rebuild?

In the spring of 2023, I put an open-source project called GPT Engineer on GitHub. You described the software you wanted, it asked clarifying questions, and then it wrote the whole codebase. Later that year I wrote down the plan for Lovable: people who couldn’t code would create, deploy, and improve software through chat and instantly see the result. More than 100,000 pull requests in our codebase later, my co-founder Fabian and our team of engineers, designers, and product managers have turned that plan into Lovable, and our users have created more than 70 million projects on the platform to date.

I recently reread my original plan, and it's eerily similar to what we ended up building. What I didn’t anticipate was how many times we would reinvent the way we get there. Along the way, we had to rethink how much freedom to give a model, how to catch its mistakes quickly, and even how to fork the computer it was working on.

I suspect other teams building software products are working through many of the same challenges. I hope sharing how we approached them makes that work a little easier.

## Narrowing the problem gave us a head start

We chose to help people create new applications. They would bring an idea, and Lovable would generate the codebase and host it for them with one click. Focusing on greenfield projects spared us the enormous variety of existing codebases, dependencies, and environments. We narrowed the problem further by choosing an opinionated stack, dropping an early selector that let users choose between frontend stacks.

Together, those choices gave us control over the libraries we used, the structure of the code, and the environment it ran in. We could run the generated code in an environment we understood, inspect failures, and improve the entire system automatically. Improvements to that common setup could help across many projects.

The principle was simple: constrain the problem, control execution, and learn from the results.

## Match the agent’s freedom to the model’s ability

Getting an application onto the screen was one thing. Keeping it working as someone asked for change after change was harder.

A developer can inspect a broken application and repair the code. Someone who has never programmed needs the product to handle much more of that work for them.

I kept pushing us toward autonomous agents because I expected models to improve quickly. I wanted us to build for those future models. Our engineer Mårten Wiman took this to a very sophisticated async design where the agent could create tasks and delegate to subagents. But less-constrained agents would get stuck in technical rabbit holes, making errors and taking actions with no predictable finish time. Giving them more freedom made the product less dependable.

So we took a step back. We deleted almost all agent code and went back to one user request resulting in a rather predictable chain of tools, with chat and a live preview on the output. By late 2024, the main editing flow was a single response containing the requested file changes. Native tool calling wasn’t reliable enough for us then, so we had the model wrap code in XML-like tags such as `<lov-write>` and parsed the edits after the agent was done.

We had retries and error handling, but no open-ended loop in which the model ran a tool, inspected the result, and chose its next move. The person could check the result in the preview and steer the next change.

The models also had limits on how much code they could consider at once. As applications grew, choosing what to show them became a problem of its own. We used fine-tuned tiny LLMs to identify the code each change depended on, including files that wouldn’t need editing, and passed that context to the coding model.

We were learning to shape both what the model could do and what it could see.

## We planned to train models. Knowledge files + RAG worked better.

Choosing the right code was only part of the context problem. As people built, we saw the same mistakes reoccur. The 2023 plan said Lovable would be powered by fine-tuned models, trained on the data the product generated, so every failed attempt would make the system better.

In 2024, we found a simpler way to start: engineers inspected chat logs from broken apps and wrote short notes explaining recurring mistakes and how to avoid them.

We called these “knowledge files.” An icon problem could become an instruction as simple as this:

```
# Icons
Only use icons that exist in lucide-react.
Never invent an icon name. If unsure, pick a generic one.
```
*Illustrative knowledge-file example.*

A retrieval step selected the relevant notes and added them to the model's context. No fine-tuning, just text. The pattern will be familiar to anyone using agent skills today: keep useful instructions in files and load the ones relevant to the task.

I believe that work was critical to our take-off in late 2024 and I even referred to it as a scaling law. A useful lesson from one broken app could help the next person without waiting for us to train a new model.

We later applied the same idea in Lovable Stack Overflow, which retrieved problems and solutions for the agent. An early version reduced the rate at which users got stuck by 5%.

## Faster agent feedback loops created a snappy experience

An agent that can correct its own mistakes still has to make you wait for those corrections. If every attempt requires a full build, even a small change can become a slow conversation.

We built sandboxes that retained each project’s files and kept its development server running while the user worked. Edits could update the live preview without a fresh production bundle.

Separate incremental TypeScript checks reused work from earlier checks to catch errors in how the code fit together. The agent could use those errors to work on a fix while the project was still running.

We already had ways to return errors to the model and retry. The point of this infrastructure was to shorten each edit-check-correct cycle. A coding agent’s speed depends on how quickly it can check and correct its work, too. These speedups for every edit made a big difference in the overall feel of Lovable.

## New foundational building block: forking and merging the computer

Keeping the development server running shortened the edit-check cycle. Later, we took that further: letting an agent return days later and pick up the same working environment in seconds.

Saving the source files wasn’t enough. We needed to preserve the running processes, too. Our stateful sandboxes checkpoint the machine’s memory, CPU state and disk, then restore it when needed.

This work became foundational to many more technical features of Lovable. It now lets us fork not just code, but an entire application with its working environment and database, giving agents a way to try different approaches from the same starting point.

What started as a way to make edits faster is now letting us ask agents to take on much more.

## The next 100,000 pull requests

The product we wanted to build in 2023 has now helped people build more than 70 million projects, from websites to highly sophisticated fullstack applications. Getting here meant reinventing how we built Lovable, multiple times. Each of those choices brought the same ambition closer: someone with an idea could make it reality without barriers holding them back.

That was the easy part.

Three problems are still wide open.

**Putting the drudgery on autopilot.** Building an app was always just the beginning. Once real customers and real money depend on it, every change gets harder and riskier. The harder problem is to own that whole lifecycle: testing, deploying, and maintaining production software, even while you sleep. Scaling, uptime, security, fixing bugs before your customers find them. The hardest part is trust. One misunderstood request from a user can delete data or break something that can’t be undone. We’re responsible for preventing that.

**Make building all types of software seamless.** Customers should just describe what they need and it works, regardless if it’s desktop, mobile, complex workflows, realtime data processing, or global business systems. Each brings different environments, constraints, and ways to fail.

**Build a better software stack.** We don’t see it as our job to just make building software seamless for our customers, it’s our job to iterate toward a better software stack altogether. This means inventing new foundations. Innovations in observability, forking and merging computer snapshots, new sync engines, durable execution patterns, verifiable security and privacy guarantees for users. There is a lot to improve before the software stack people build on evolves to its ultimate, most helpful shape.

Solve the above three and you get something more than an “AI Software Engineer”: a platform people run entire businesses on. Our users are already pulling us there: they connect their Slack, ingest their data into Lovable, connect Lovable to everything else they use. They run their companies, use Lovable to handle their payments, and then they use features like our growth tools, discoverability, and analytics to grow their businesses with us. They even ask Lovable to incorporate their companies. Our job is to do everything we can to unblock their ambitions.

That is the work cut out for us in the next 100,000 pull requests.

If you want to work on these hard problems, where the answer isn’t in a textbook and the result shows up in front of millions of people the same week, come build with us at Lovable.

And if you’re working through the same problems elsewhere, subscribe to our newly launched tech blog (did I just write “tech blog” in 2026?). We’ll share what we learn working toward our vision directly from the engineers doing it.
