---
url: https://shiftmag.dev/review-fatigue-12276/
date_fetched: 2026-10-03
---

# Review fatigue is real. Here’s what my team did about it

I have never particularly liked being the code reviewer.

It feels like getting all the responsibility but zero fun of actually writing the code. Even before most of the code became AI generated, the paradox was familiar to many: the more lines of code a PR has, the less likely it is that the changes will get a very detailed review. **Why do we expect standard code reviews will still work when AI generates much more code volume?**

For a workplace where agents are doing the heavy lifting, benefits of code review through classic PRs are gone, and there are a few reasons why:

- Engineers do more **prompting and decision**making than coding. But most people aren’t reviewing the former.
- The **reviewer might be the first one to actually have to read all the code**. Since the implementer gets familiar with the implementation through the prompting process,**the reviewer is in a worse position**. If they go through everything line by line, it probably takes them longer than it took the original developer. If the real time cost of review isn’t reflected in sprint capacity planning, reviewers are probably not given enough time to finish the task.
- **AI code review agents are a logical next step.**To some extent, they are already utilized by AI-transformed companies. However, just throwing an agent at a PR without any structural and conceptual changes around code reviews, such as new types of safeguards and a robust production setup with automated rollbacks, misses the point of code reviews entirely. The human is no longer in the loop.
- **Context switching.**It was always an unwanted companion of software engineers, but now it’s like a shadow we can’t escape. While our agents are thinking and combobulating, we do something else. We spin up a few agents for a different task, and then another for the next one. What happens to the PRs then? They pile up. The pressure to downsize that pile can be real, so the review quality becomes questionable.

Teams have adapted to AI quickly in many parts of the workflow, but code review is still lagging behind. Instead of following the same workflow as before, we should rethink what code review is and how we practice it. While this might not suit best for every organization, for me and my team, the most useful model right now is a pair-programming spinoff: **pair planning and pair validation**.

**Pair planning**

Pair planning is the first step. It starts with prompting the agent. And it matters how you do it.

The easy option is to drop the task description to an AI agent and let it do the work however it wants. Long-term, this is a good way to create a big pile of slop.

A better way is to ask the agent to **propose multiple options and document them**, without any code changes. While the LLM is doing its magic, this is your time to take a moment and think about the task in front of you. Ideally, the thinking happens out loud because you’re pairing with another colleague.

Why is the part when you’re thinking without any AI assistance important? Once you start reading the AI output, it’s highly likely your brain will lock in. Alternatives which the agent didn’t propose are more difficult to think of. It’s also possible that the task description wasn’t detailed enough and the agent didn’t take an important detail into account. Any caveats, edge cases, deployment scenarios, or other details that come to mind while you’re brainstorming about the possible task implementation details are good to note down before you read the agent output.

The agent is done planning. Output is ready. **Now what?**

Both you and your pair should get to reading. Flag any issues you see and discuss them. Refine the plan. Squeeze the implementation details from the agent. Be aware of all components that will change and how. This is the juicy part of the task.

With AI added to the traditional code review flow, we often plan the implementation twice. The first time is by the original engineer implementer. This person will make all the decisions individually. They will guide the implementation. Then the second time is by the reviewer, possibly in even more detail. If the reviewer doesn’t agree with the way task was implemented and proposes a different approach, the author and agent will go back to the beginning. All tokens used on the wrong path were wasted.

These kind of “disposable code” scenarios are happening more and more. **When pairing with another colleague during the planning phase, it’s more likely you will choose the best implementation option and reach your goal in minimal time and tokens spent.**

**Structural prompt committing**

Most of the engineering work gets done in the planning phase. Decisions are made and propagated into code. From commits only, we can only see *what* was decided. *Why* it was decided and *what the alternatives were* is valuable information that tends to get lost.

Prompting practices can be refined to avoid losing decision data. I practice structural prompt committing. In the codebase, each task is documented with my prompts, distilled versions of agent responses, and a few more useful files. The structure can look something like:

- task folder
- **prompt.md**- chronological log of your prompts and distilled info about agent responses, append only
 
- **context.md**- current context, changes can be traced through git versioning
- either for human reference or for loading agent context
 
- **plan.md**- after all the decisions are made via prompting, full plan is documented to a separate file
 
- **summary.md**- summary of implementation changes
 
 

This structure is useful while working on the task, especially if multiple persons are involved. Afterwards it can be deleted. For important decisions, architectural decision records should be made.

**Implementation ideas**

How you decide to do the implementation after the plan is ready depends on many factors. Some ideas can be applied generally:

- If unsure between options proposed by an agent, **ask it to prototype them on multiple branches**. See how they act in action and validate through tests, metrics, etc.
- Small commits are desirable. It is easier to follow the changes and revert precisely if needed.
- For bigger tasks, create checkpoints. Don’t let the agents create a massive change which will have to be fully reverted if it proves wrong.

You might be wondering, **what should you do while Claude is claudeing**? Should the pair of colleagues just… wait and watch?

Well, maybe. If the checkpoints are small enough it might make sense to wait and discuss the direction the agent is taking. If not, it could make sense to split up and meet again when the project is ready for validation and review. It’s up to you to weigh if the context switch is worth it.

**Pair validation**

Whatever road you choose, you should arrive at the last part: pair validation. Why didn’t I call it pair review? Because review sounds like you’re just reading the code. Instead, at this stage you should validate, i.e. prove, that code works.

If the implementer and reviewer validate alone, the job is doubled and less efficient. **Since AI is generating most code, we are all validators more than implementers. **Let’s not validate twice. Two colleagues should go through the code changes in a structured way, making sure the requirements and acceptance criteria are met. Ideally, start from the tests. Are they proving the new code is working? Are they covering all use cases? Are they guarding the future implementation from possible breaking changes?

Through conversation, concerns can be raised and resolved immediately. Learning and knowledge sharing are instantaneous. Don’t lose this value by spending time navigating in heaps of generated code and writing async comments someone should check between three different context-switching sessions.

If you work in an environment that allows it, I encourage you to start thinking about the process of implementing and reviewing as one unit, not two separate jobs for two separate people. A good way to avoid getting in the “just review this quickly please” trap is to plan ahead. During planning of what each teammate will do in a given period, allocate the same amount of time and resources for both the implementer and reviewer. **Teamwork begins before the task is started.**

**Not all jobs can follow this framework.** Examples are open-source project PRs, distributed teams across multiple time zones, and contractor engagements. In these kinds of environments, reimagining how we do reviews gets even more interesting. I expect new and completely different frameworks to emerge. Until then, my team and I are quite confident shipping pair-planned and pair-validated code.

Until then, my team and I are quite confident shipping pair-planned and pair-validated code.
