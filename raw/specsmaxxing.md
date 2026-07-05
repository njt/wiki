---
url: https://acai.sh/blog/specsmaxxing
date_fetched: 2026-07-05
backfilled: true
---

## Does this look familiar?

Wow. Claude. Mind-blowing. The whole feature works great. But I forgot to mention one**very**important edge case.

Ah, and I just noticed. You usedYou’re absolutely right! Let me fix that.

*offset pagination*for the table UI. Obviously

*cursor pagination*is a better fit here?

Also, is that an N+1 query? Fetching for every row in the table? Why not do a single round-trip?You’re absolutely right! Let me fix that.

This is why I still have a job, right?You’re absolutely right! Let me fix that.

…

### Peak Slop

I’ve watched this scene play out many times, but the frequency is decreasing. Both my tools, and my methods for using them, continue to improve. I think Peak Slop has already come and gone.**the sky is not the limit. The context window is the limit.**And what happens when I fill the context window? Or kill a session? Switch machines? Hand off the project to someone else? We already know what happens. The agent goes off the rails, or requirements get lost, and critically important detail gets squashed. So we adapt and mitigate. We document. We list requirements. Yes, millions of us are coming to the same realization: we should put more requirements in writing. We should update those requirements when they change. Look! I wrote a spec! Am I doing

*Perhaps, but it is nothing new. Our mentors tried to teach us these habits decades ago.*

**spec-driven development?**## Specifying the plane while we fly it

What’s your favorite flavor of spec? A`README.md` and `AGENTS.md` is a good start. Don’t forget a `testing-guide.md`. Maybe an `architecture.md`, a `PRD.md`, and a design doc too. Have you considered `md.md` (to teach your agents how to write `.md`)? The more `.md` the better, right?
Unironically, yes. Docs and unstructured specs can get you very, very far. Much farther than prompts alone. If you aren’t writing *any*docs yet, you should just stop reading this and start there. And remember, slop in, slop out. Nothing beats an organic, pasture-raised,

**hand-written**spec. Spec-writing is where the act of

*software engineering*really happens. So a few weeks ago, I started asking myself, how far can I take this? How far

*should*I take this?

## Dreaming in markdown

As the story goes, I fell into an AI psychosis, I became a “spec maxxi”, and I spent hours and hours writing the most beautiful PRDs and TRDs you’ve ever seen. I drafted templates and skills and roles, thinking that maybe my agents can write specs too! I assembled an army, working together like a mini*dark factory,*to turn my specs into reality. My tasks grew more ambitious, and at one point I broke the vibe-coding sound barrier:

*an agent that ran for 1.5 hours unsupervised!*Exciting. But what did that army ship for me? Well, it wasn’t slop, in fact it

*worked*, which is more than I can say about the garbage that other companies force me to use every day. But it was still a bit sloppy. I’m far from a perfectionist and I love cutting corners more than most, but this somehow wasn’t good enough. One hallmark symptom of AI psychosis is

*using AI to build AI harnesses for building products*, rather than just using AI to build the damn product. I embraced my illness, threw out the branch, scrapped all my markdown, and started all over again.

## Acceptance Criteria for AI (ACAI)

A few days later, I noticed an ambitious little sub-agent doing something unexpected.**The little guy just went and numbered my requirements and then referenced them all over my codebase.**Why? I did not ask for this! I was disgusted. This is a tight coupling of code to spec, and spec to code, which is bad right?

*You really expect me to refactor all my code every time I change my spec?*Oh. I suppose that’s a good thing? Interesting. I wonder…

- Perhaps these tags can help me navigate these massive PRs?
- Perhaps they can point me to **where, exactly, a requirement is satisfied or tested!**
- Perhaps I can annotate them with **notes and states (todo, assigned, completed)!**
- Perhaps I can start tracking **acceptance coverage**instead of test coverage!

**ACIDs (Acceptance Criteria IDs).**But a few questions remained.

- *Can my ACIDs number and label themselves?*
- *Is it cumbersome to keep them aligned?*
- *How do I share specs and progress across sandboxes, branches, features and implementations?*

## Acai.sh - an open-source toolkit

I built Acai.sh to solve some of these newly invented problems. And I’m very excited about the results.- A simple and flexible template for feature specs, called `feature.yaml`. Feature.yaml makes it possible to reference each requirement by ACID.
- Tiny CLI to power your CI and your agent (available on npm or via github release).
- Webapp that serves a dashboard, and a JSON REST API (Elixir, Phoenix, Postgres).

**free for a while, or maybe forever**depending on how popular or expensive this gets. The source code is on GitHub under an Apache 2.0 license.

## How it works

### Step 1 - Specify

Start by writing a spec for a feature. Be ambitious— something that adds real value. Don’t put nitpicky UI and nail polish stuff in your specs. Keep the requirements concrete, testable, and focused on what really matters (functional behavior + critical constraints). Rather than markdown, use acai’s`feature.yaml` format instead. A spec in Acai is just a numbered list of requirements.
feature.yaml

`my-feature.ENG.2`.
### Step 2 - Ship

Copy and paste the prompt below.### Step 3 - Review

No more file-by-file GitHub PR reviews. Use the Acai.sh dashboard to review*requirements*instead. Ideally, you just add

`acai push` to a GitHub action (example CI/CD workflows coming soon).
- Create a free Team and Access Token at https://app.acai.sh
- Expose the environment variable
- Push specs and code refs to the dashboard for review

### Step 4 - Iterate / repeat

The goal is to work spec-first. Change the spec itself, or use the dashboard to attach notes to the spec. With the right harness, you can get your agents to react and self-assign (using`acai` CLI commands).
The result should be less time spent prompting and re-prompting, less time reviewing, less sloppy code generation, and far more time thinking about what you want your product to be.
`plan → implement → review` loop. That part is not included (yet). I plan to share some powerful examples of that soon.
## Future Gazing

Even coming out my LLM psychosis, I’m still finding this approach useful and productive. I believe I’ve found the sweet spot between rigour and vibes, structure and flexibility. Maintaining an itemized list of acceptance criteria encourages me to step back and focus on the things that matter, and to rethink how I test and validate my output. Again, none of these ideas are really new. But I feel the gravitational pull, and I’m wondering where it all leads, and what comes next.### Thought Experiment

Imagine your entire application, however complex, was generated instantly*the moment your fingers started typing in the prompt window*. Imagine that magically, the same prompt input always created same deterministic output, and cost you nothing, and was ready in milliseconds.

*somewhere*, even if you don’t write it down. The spec is what you

*want*the software to be. It often exists only in your head or in conversations. You and your team and your business

**will always care what the spec says**, and that’s never going to change. So you’re better off writing it down now! And I think that a plain old list of acceptance criteria is a good place to start. (That’s really all that

`feature.yaml` is.)
A few more things follow from there;
### -> From Specsmaxxing to Testmaxxing

When code is generated faster than you can read it, the bottleneck moves to QA and validation and your*confidence*that the code is to-spec. So investing heavily in QA automation and test coverage has a massive ROI. And after that, after maximal test coverage and observability has been achieved, our attention shifts away from the diffs in GitHub; we need a place to relate the specs to the important artifacts (the implementations, the QA feedback, user feedback, builds, runs, reviews and reports). Maybe Acai.sh can become that place. For now, it is only focused on the specs and their implementations in code.

### -> From Testmaxxing to reactive software factories

The final unlock is when your LLMs can react to a red test or alert. If the specs are well defined, and you have high confidence in your integration pipelines and QA, then the LLM is fully empowered to react without the need for you to intervene. It seems every well-funded software team on earth is currently busy building their own bespoke solution for this and I’m excited to see what works and what doesn’t. As a first step, I put some OpenCode Agent templates on http://acai.sh that have been working well for me so far.## Comparison to other spec-driven development tools

Embarrassingly I did not know about any of these until I was almost finished building my own. The key differentiator is that Acai.sh is**focused on helping you track**, which is a slightly different problem than what other tools are trying to solve.

*acceptance coverage*and*spec alignment*across many implementations*I can not offer unbiased feedback, because I suffer from Not Invented Here syndrome. It may even be the case that the Acai.sh approach actually enhances these tools.*

### GitHub SpecKit

SpecKit reads to me like ‘vibe coding with extra steps’, i.e. a CLI that augments your agents with more prompts and more skills. This is surely productive, but it is solving a slightly different set of problems.**OpenSpec**

The first thing I noticed when reading their docs, is their stated mental model, which is written: *“Specs […] describe how your system currently behaves.”*. I fundamentally disagree with this. In Acai.sh, specs describe how your system

**SHOULD**behave. Current behavior is transient. In addition, their process appears to lean into “AI generated spec writing”, which has never gone well for me Lastly, the process of versioning and diffing specs is just basic gitops, and I don’t need a CLI for that. Same goes for the agentic ops and task creation (Vanilla OpenCode does just fine, thanks).

### Kiro

I don’t like EARS syntax, but I don’t like unstructured markdown either, and Kiro claims to convert the latter into the former. Acai is different— I came up with feature.yaml to strike a balance between unstructured markdown and cumbersome EARS / gherkin. Unlike Kiro, Acai.sh does not try to solve end-to-end delivery either (yet).### Traycer.ai

Looks like a very useful product for managing artifacts and spawning agents to implement them. But they are still plain-old .md files. Acai is more opinionated about how you should write the spec (feature.yaml).## Reasons you might not like acai.sh

Like any tech, it comes with tradeoffs;- You might not need to write specs. If your product is low-stakes, or simple, just keep prompting.
- Acai specs are opinionated. **You should write 1 spec per feature**(though the feature boundary or slice is up to you). My opinion is that cross-cutting feature specs are easier to iterate, and really shine when a feature touches many codebases (frontend / backend / microservice etc.) and has many implementations (Production, Staging, Fix, etc.)
- ACIDs rely on **stable numbering.**This creates zero friction when*drafting*a new spec, but requires care when revising a spec: you must re-align the code whenever your spec changes (almost forgot that this is the whole point?). The feature.yaml syntax supports`deprecated`and`replaced_by`flags as well, if you want to maintain a complete spec history inline.
- Acai **discourages**you from putting design and superficial requirements in your specs. Specs are for behavior, and constraints, and nothing else. I’ve learned from experience: get it working to-spec first, then do the superficial nail polish last 💅🏼.
- Acai requires you to adopt the **feature.yaml**format. Almost zero learning curve. I have written an introductory guide to writing them. I recommend reading that first. Use`npx @acai.sh/cli skill`to teach your agent how. I also wrote a spec-for-the-spec if you want to go deeper.
