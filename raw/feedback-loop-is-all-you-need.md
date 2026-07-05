---
url: https://zernie.com/blog/feedback-loop-is-all-you-need/
date_fetched: 2026-07-05
backfilled: true
---

## Contents

- I can't use scheduled agents on my real codebase
- The old loop vs the new one
- The real enemy: tech debt you can't see
- Your CLAUDE.md is a suggestion. Your linter isn't.
- Guardrails: complexity is the killer constraint
- What CI actually catches
- The investment
- The organism
- Start here
- Closing
- Join the Discussion

## I can't use scheduled agents on my real codebase

So Claude Code added scheduled agents recently. Recurring tasks, native, built right in. The thing we've been dreaming about since the first AI coding demos. Schedule an agent, go to sleep, wake up to merged PRs. An engineer that works while you don't.

And I'm sitting here like… I can't even use this. Not on the real codebase. Not at work. I was at archive.com for years. We'd gone through three design systems. Started on Shopify's Polaris, switched to Ant Design when we outgrew Shopify, then migrated to shadcn/ui and Tailwind because Ant Design became its own kind of legacy. Years of that, three UI frameworks, conventions that lived in people's heads, business rules no one ever wrote down. You point an agent at that, it'll run. It'll produce code. Beautiful, idiomatic, unholy code that imports from all three design systems in one file and somehow passes every check you have.

So what do you do? You can't review everything. You can't slow down the agents. And you definitely can't trust them to just figure out which design system to use.

So this is what I figured out. I use Claude Code, so the examples come from that world, but honestly it doesn't matter if you're in Cursor, Copilot, Codex, or Devin.

## The old loop vs the new one

**The old loop:** write or review code, spot smells by experience, leave comments explaining intent, promise to fix things "later" (which usually meant never).

**The new loop:** encode rules once, let agents iterate against them, observe what fails, tighten the constraints. Less "remember this next time," more "this literally cannot happen."

Took me about a week to feel this. When code can be produced nonstop, you're the bottleneck. Not the agent. You.

## The real enemy: tech debt you can't see

Here's what nobody warns you about: the scariest thing isn't when agents break your code. It's when they don't. Code that compiles, passes every test, looks totally fine in review, and quietly violates architectural assumptions you thought were safe.

I watched this happen. The agent adds a form to a page I'd already migrated to shadcn/ui. It reaches for Ant Design's `<Form.Item>` because that's what the other form on the page still uses. Compiles fine. Renders fine. My migration just went backwards by one component, and nothing in the pipeline noticed.

Same thing with CSS. Agent writes a new component using Tailwind utilities, correct per our current standard. But it copies a padding value from an old Ant Design component next door: `p-[24px]` instead of our spacing scale `p-6`. One magic number won't kill you. Fifty will. Each commit looked fine on its own. By the time you notice, you're three weeks deep in inconsistency.

A human would catch this. You'd look at the import and go "wait, why are we pulling from Ant Design here?" Agents don't have that gut feeling. Without hard signals, you're just sending "still broken" for the fifteenth time.

*Source: ProgrammerHumor.io*

That's when I started obsessing over feedback loops. How fast can the agent find out it's wrong?

## Your CLAUDE.md is a suggestion. Your linter isn't.

My first instinct was the same as everyone else's. Write better instructions. CLAUDE.md. Skill libraries. Document annotations. Write it down clearly enough and the agent will follow, right? Even Vercel shipped a skill library — 40+ React performance rules, beautifully written, structured as SKILL.md files for AI agents. Really solid stuff.

But instructions alone? Yeah, no. Not when the agent is shipping more code than any human can review.

Look, instructions matter. A good CLAUDE.md cuts iteration cycles way down. But there's a difference between "helps" and "guarantees," and CLAUDE.md is firmly in pirate code territory. We're basically betting the codebase on a probabilistic system getting lucky every single time. Sometimes it does. Sometimes you get beautiful, idiomatic code on the first try. And sometimes it drops an unindexed query into a hot path and you find out when the database melts at 2 AM on a Friday.

And then it hit me... we already solved this problem. Decades ago. Unit tests exist because humans forget edge cases. Linters exist because humans can't agree on where to put a curly brace. CI exists because "it works on my machine" stopped being funny after the third production outage. We've known this for years.

And yet here we are, doing the exact same thing with LLMs. "Just write a really good CLAUDE.md." "Just add more skills." Remarkable how quickly we forgot all of that.

CLAUDE.md explains the why and helps the agent get it right on the first try. A lint rule makes sure it can't get it wrong. Skills speed you up. Linters keep you honest.

If you can only have one — take the linter.

## Guardrails: complexity is the killer constraint

This is what runs on every change in my setup:

- ESLint, because the agent doesn't have ten years of muscle memory about your import conventions
- SonarJS to kill entire bug classes before they start
- Strict TypeScript (if the types are loose, the agent will find every crack)
- Opinionated React constraints, so there are no "creative" component patterns at 3 AM
- Prettier, because formatting debates are over

But honestly? The single most effective thing I've done — and I tried a lot of things — is brutally strict complexity limits. Not style rules. Hard caps on how big and how tangled code is allowed to get.

Turns out ESLint already has rules for this. `complexity` caps cyclomatic complexity. `max-depth` limits nesting. `max-lines-per-function` forces decomposition. `max-params` keeps interfaces narrow. `max-statements` stops functions from doing twelve things at once. SonarJS adds `cognitive-complexity`, which is smarter about nested conditionals. I wish I'd turned these on years ago.

I learned this the fun way. Without complexity limits, an agent will cheerfully generate a single 150-line function with six levels of nesting, three early returns, and a switch inside a try-catch inside a loop. Compiles. Passes tests. Even works. Then the next agent touches it, makes it worse, and congratulations, you now have two 150-line functions. Set `max-lines-per-function: 40`, `complexity: 10`, `max-depth: 3`, pair with `--max-warnings=0` and watch what happens. The agent *has* to decompose. It starts extracting helpers, naming things properly, separating concerns. Almost like it knew how to write clean code all along, it just needed someone to insist. The specific numbers matter less than having them at all. Start strict, loosen only when it actively hurts.

I keep saying ESLint because that's my world, but the stack genuinely doesn't matter. RuboCop, Ruff, clippy — same principle. Off-the-shelf linters handle syntax though. Your architecture? That needs something custom.

One thing that surprised me: before writing anything custom, I found a ton of value just turning on plugins we'd never bothered with. SonarJS, unicorn, perfectionist. They'd been around for years, we just never got around to adopting them. The usual excuse was "too many existing violations to fix." With an agent, that excuse is gone. Point it at the violations, let it triage them in bulk. Five minutes, a `recommended` config, entire bug classes gone.

### Can this be a lint rule?

At some point I caught myself asking: can this be a lint rule? Every time something went wrong. It became almost automatic. And once that switch flips in your head, you're not a reviewer anymore. You're the person who makes sure the mistake can't happen again. Ever.

First one's the worst. After that they snowball. Real example: our agents kept cheerfully adding `console.log` to production code instead of the custom logger that routes to Datadog. Every. Single. Time. A 10-line lint rule fixed it. Forbid `console.log`, suggest `logger.error`. Done. Never thought about it again.

People hear "50 custom rules" and freak out. I get it, I did too. Some of those rules will be wrong. But here's what happens: someone hits a bad rule, gets annoyed, opens a PR to change it. And suddenly you're having the architectural conversation you never had before. That PR is worth more than the rule itself. And when a rule requires migrating existing code, AI + codemods make the cleanup feasible in hours rather than quarters.

Linting is one side of it. Testing is the other, and honestly AI has made tests embarrassingly cheap to write. The kind of thorough, exhaustive test suites I always said I'd get around to writing? An agent bangs those out in minutes.

Only problem? Rules only catch what you've already seen. The stuff that hasn't bitten you yet is still out there.

## What CI actually catches

Every push triggers the full check. Nothing personal against the agent — I don't trust *anything* that hasn't been verified. Including my own code, for what it's worth.

Playwright screenshot tests were a game changer for me. The stuff they catch is wild — a z-index regression that buries a modal behind an overlay, a layout shift from a refactored flex container, a button that renders but is completely unclickable. None of that shows up in unit tests. Chromatic does the same for Storybook-based workflows. If no one looks at the screen, the screen will break, and boy did I learn that one the hard way.

I slept on property-based testing for years. Instead of writing individual test cases, you define a *property* that should always hold: "this function should never return a negative number" or "encoding then decoding should always return the original input." The framework generates hundreds of random inputs and tries to break it. Incredibly effective, but I never adopted it because writing good property definitions was tedious. AI flipped that. Now I just point an agent at my code and it figures out what properties should hold. One team ran agents against 933 modules and got 984 bug reports, 56% valid, roughly $10 per real bug found.

Security is the one that scared me straight. DryRun Security tested Claude Code, Codex, and Gemini building two apps. 87% of PRs had at least one vulnerability. Not typos. Stuff like WebSocket endpoints missing auth that the REST API had, rate-limiting middleware defined in a file but never actually mounted. The agent wrote the middleware correctly. It just didn't know it wasn't running. That one messed with my head — static analysis sees the file exists but can't tell it's not wired in. You need CI that actually boots the app and checks behavior.

And then runtime monitoring. We wire Sentry and Datadog into the task queue. Something breaks at 2 AM, it becomes a task the agent picks up. I wake up to a fix, not a fire.

Yeah, that's a lot of machinery. My manager would ask: what does this cost?

## The investment

Tokens are cheap. SaaS subscriptions are cheap. The real cost is your time, and none of this ships features. I know. I had this argument with myself for weeks.

*Source: Memedroid*

But then I saw the numbers. CodeRabbit analyzed 470 GitHub PRs and found AI-generated code has 1.7x more bugs than human-written code. 2.74x more security vulnerabilities. Their words: "We no longer have a creation problem. We have a confidence problem."

Yeah, you're paying for tokens. Taxi drivers pay for gas, the car, the medallion, the insurance, on 5-10% margins, and nobody bats an eye. We're a bit spoiled in software. 80%+ gross margins, entire means of production is a laptop and a chair you already own. A $200/month tool that catches even one production bug per quarter? Mate, that's already paid for itself ten times over.

A senior engineer costs $150-200/hour loaded. A production bug found by a customer? Days of investigation, emergency fixes, trust you don't get back. Meanwhile, a custom lint rule takes an afternoon and catches that class of bug forever. A Playwright screenshot suite takes a day. I don't know why I waited so long.

And it compounds. Every guardrail you add multiplies what the agent can ship without you. One more rule, one less thing to review. Ten more rules, entire categories of bugs that just... stop happening.

Karpathy put it better than I could: "The goal is to claim the leverage from the use of agents but without any compromise on the quality of the software." That's what I've been fumbling toward this whole article.

## The organism

When I finally wired all of this together, something clicked. The system started tightening itself:

Here's what a Tuesday looks like now: agent opens a PR. Custom lint rule catches a barrel-file violation, agent fixes it. CI runs Playwright, screenshot shows a layout shift, agent adjusts the CSS. Sentry reports a spike in 404s on staging, new task gets created, agent picks it up. I reviewed some diffs over coffee. Nobody else typed a line of code.

Every bug that reaches CI becomes a rule that prevents the next one. The thing feeds on its own failures. At some point I stopped calling it a toolchain. Felt more like an organism.

And I'm not the only one who noticed. Spotify built a background coding agent called Honk on top of feedback infrastructure they'd been investing in since 2022, three years before the AI part. They're merging 650+ agent-generated PRs to production per month now. Devin's merge rate doubled from 34% to 67% when they improved codebase understanding, not the model but the context around it.

Same story both times. The model didn't get better. The loop got tighter.

## Start here

### Where are you?

| Level | What it looks like | The tell | 
|---|---|---|
| 0 — Vibes | No custom linting, no CI, you review everything manually | "My eyes are the only thing between the agent and production" | 
| 1 — Guardrails | Standard linters + CI, but no custom rules | "The agent passes lint but still drifts architecturally" | 
| 2 — Architecture as Code | Custom lint rules encoding your team's conventions | "CLAUDE.md rules are migrating into the linter" | 
| 3 — The Organism | Self-tightening loop: agent → rules → CI → observability → tasks → agent | "I schedule agents overnight and review diffs in the morning" | 

If you're at Level 0, same as me a year ago. Here's what I wish someone had told me:

**Today:** that PR comment your team keeps leaving over and over — about import conventions, barrel files, console.log in production — turn it into a lint rule. Just one. That's how it starts.

**This week:** add Playwright screenshot tests for your three most critical pages. I was shocked what they caught that unit tests missed.

**This month:** schedule an agent for something safe. Dependency updates, test suite maintenance, stale branch cleanup. Let it run overnight, review the PR in the morning. I started with Claude Code web; when that wasn't enough, a cheap VPS gave me more power for the same idea.

**How you know it's working:** you can delegate a task from your phone, review the diff on a commute, and trust the result. Not working less, just not chained to the laptop anymore.

If you want a starting point, I put together a playbook with tooling setups at different budget levels, and a companion repo — vigiles — that automates some of this. Would've saved me a few weekends.

## Closing

Three design systems in five years. The agent doesn't know which one to use unless you tell it, deterministically, on every commit.

That's the whole thing, really. That agent I couldn't point at my codebase? It wasn't waiting for a better model. It was waiting for better sensors.

LLMs are probabilistic. They'll get it right most of the time, and on a real codebase "most of the time" will eventually ruin your Friday night. I've made my peace with that. No amount of prompt engineering changes it.

So I stopped chasing clever prompts. Started chasing boring, deterministic, tedious feedback. The kind that fires whether or not I'm watching. The kind that doesn't care how confident the model was.

Linters don't sleep, and CI doesn't get tired. That's more than I can say for myself.

*The title is a nod to "Attention Is All You Need" (Vaswani et al., 2017) — the paper that introduced the Transformer architecture.*
