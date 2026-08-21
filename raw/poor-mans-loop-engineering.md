---
url: https://awaitinginput.substack.com/p/poor-mans-loop-engineering
date_fetched: 2026-08-21
---

Getting agents to perform all of your tasks for you without getting involved in their interaction cycle sounds like a fairy tale. To burst the bubble right away, agents can’t do this (yet). What they can do is take a scoped task and implement it end to end, without introducing regressions (most of the time, but even your best engineer is never perfect). Even this claim must take insane amounts of effort to achieve, right? Wrong, it doesn’t. The poor man's version requires two things: your agent needs to be able to test its own work, and it needs someone other than itself to review that work. This is the foundation of loop engineering.

**What even is Loop Engineering?**

Most of you reading this are probably already familiar with Loop Engineering, but a quick back story if you aren’t. Back in June, Addy Osmani from Google coined the term in his tech blog 1 “Loop engineering is replacing yourself as the person who prompts the agent. You design the system that does it instead”. But this thesis isn’t new. Devs have been creating loops for their agents since late 2023 to help give more and more work to the agents as the models get better. The goal isn’t quite “press go and come back tomorrow”, although that’s an easy conclusion to draw, the goal is to ask what happens when you automate some of the iteration cycles we normally perform as software developers instead.

## The poor man’s loop

#### Test Test Test

The “loop” isn’t that hard to get started; making it truly autonomous is. The first step is, in my opinion, the most important one, give the agent a way to know it’s wrong. Testing has forever been integral to software development, and it’s always been a place developers try to skip. Well, no excuses anymore with AI to help write the tests for you! Having said that, you don’t need to hop on VSCode and churn out PyTests, that’s not the point, the point is setting up a safe way for an agent to run integration/E2E tests (they already write a boatload of unit tests so don’t worry about that, they got it covered). This is critical to the loop. Spend time making the system accessible to an agent, be it solidifying Make commands to reliably spin up the Docker stack or hooking up a UI sandbox for the agent to click around and gather screenshots. You need a way for the agent to know if what it’s doing is right. With a true testing environment, we can get closer to those sweet, sweet benchmarks the frontier labs love to gloat about: in this sandbox, here are the tools, go do the thing.

```
# Start the local environment
up:
  docker compose up -d
# Run unit + integration tests
test:
  pytest tests/
# Run end-to-end tests
e2e:
  npx playwright test
# Run linting + type checks
check:
  ruff check .
  mypy .
# Give the agent one command for everything
verify: test check e2e
```
#### Get some more eyes on this

Next comes a critical feature in helping the agents bridge a gap in their current intelligence: adversarial reviews. Getting two fresh isolated context sessions using cross model validation is a method I have adopted. I’m not alone here. This was a method used in the controversial post “Rewriting Bun in Rust”2 to help migrate thousands of lines of code. Simply set up a hook in Claude to use fresh agents to perform a review of your current branch/worktree changes against the goal that you set for the session. The reviewers get a clean view of what should have been done and what was actually done. *Do the tests pass? Are there gaps in the initial vision that weren’t implemented? Are there clear issues that might arise with scaling or security?* They can step through a simple process to perform all of these checks and report back if it is clean to put up a PR. This adversarial review is critical. We use two agents (and, for best coverage, cross model validation 3) because agents aren’t deterministic, and we want to cover as many possible paths, thought processes, and high-level failure points as possible.

```
## Adversarial Review
When a task is complete, do not immediately mark it as finished.
Run an independent review using fresh agent context.
Review the implementation against:
- The original task or specification
- The current git diff
- Test results
- Relevant changed files
The reviewer should look for:
- Missing or partially implemented requirements
- Logic errors and regressions
- Security issues
- Edge cases
- Weak or missing tests
- Unnecessary complexity
The reviewer should actively try to find reasons the change should not be merged.
Return:
- PASS if no material issues are found
- FAIL with specific findings if problems remain
If the review fails:
1. Evaluate the findings
2. Fix valid issues
3. Re-run tests
4. Run the review again
```
## That’s it?

```
# Task Loop
Given a task and specific goal continue implementation until done.
## Definition of Done
Before declaring any task complete:
1. Run the project's tests and fix any failures.
2. Verify the feature end-to-end using the available tools.
3. Ask two fresh, isolated agents to independently review the changes against the original task.
4. Give reviewers the original task, git diff, and test results.
5. Reviewers should actively look for missed requirements, bugs, regressions, security issues, and weak tests.
6. Fix valid findings, then test and review again.
Repeat until tests pass and no material review findings remain.
Do not mark the task complete until the loop is clean.
```
Congratulations, you now have a loop! Your agent is armed with the tools it needs to complete its tasks, so given a clear and isolated spec you can expect the agent to get you most of the way there. As you iterate and expand the skills, the loop will get better and better, letting you automate most of the tedious implementation work. But don’t run too far away, Addy Osmani himself warns against this. Stepping inside the loop is a critical part of the process. We are shifting our work towards setting up automation, but we are still required to be the engineer and own the system at the end of the day. And be warned, the agents lack taste. While you churn out tickets for your agents, make sure to take a step back, and think if this really makes any sense to do in the big picture.

Love the post - really resonate with the idea that the goal isn’t to engineer ourselves out of the loop, just out of the most tedious parts of it. Let the agents do the write-test-fix merry-go-round while we keep the judgment calls. The testing + independent review setup feels like a very practical way to make agents actually useful beyond “look, it generated some code.” 👏
