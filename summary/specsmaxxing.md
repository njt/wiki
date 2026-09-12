---
url: https://acai.sh/blog/specsmaxxing
title: "Specsmaxxing"
author: unknown (acai.sh blog)
date_fetched: 2026-05-14
date_published: unknown
topics:
  - agent-coding-workflow
  - specifications-as-the-product
---

# Specsmaxxing

On overcoming AI psychosis, and why I write specs in YAML (plus open-sourcing a toolkit for you to try)

## Does this look familiar?

Wow. Claude. Mind-blowing. The whole feature works great. But I forgot to mention one **very** important edge case.

*You're absolutely right! Let me fix that.*

Ah, and I just noticed. You used *offset pagination* for the table UI. Obviously *cursor pagination* is a better fit here?

*You're absolutely right! Let me fix that.*

Also, is that an N+1 query? Fetching for every row in the table? Why not do a single round-trip?

*You're absolutely right! Let me fix that.*

This is why I still have a job, right?

*...*

## Peak Slop

I've watched this scene play out many times, but the frequency is decreasing. Both my tools, and my methods for using them, continue to improve. I think Peak Slop has already come and gone.

We are entering the post-slop era. My software is more robust, better tested, better integrated, and more observable than ever before. And my velocity keeps increasing!

Some days it feels like the sky is the limit. Other days, I am painfully reminded, **the sky is not the limit. The context window is the limit.** And what happens when I fill the context window? Or kill a session? Switch machines? Hand off the project to someone else?

We already know what happens. The agent goes off the rails, or requirements get lost, and critically important detail gets squashed. So we adapt and mitigate. We document. We list requirements.

Yes, millions of us are coming to the same realization: we should put more requirements in writing. We should update those requirements when they change. Look! I wrote a spec! Am I doing ***spec-driven development?***

Perhaps, but it is nothing new. Our mentors tried to teach us these habits decades ago.

## Specifying the plane while we fly it

What's your favorite flavor of spec?

A `README.md` and `AGENTS.md` is a good start. Don't forget a `testing-guide.md`. Maybe an `architecture.md`, a `PRD.md`, and a design doc too. Have you considered `md.md` (to teach your agents how to write `.md`)? The more `.md` the better, right?

Unironically, yes. Docs and unstructured specs can get you very, very far. Much farther than prompts alone. If you aren't writing *any* docs yet, you should just stop reading this and start there.

And remember, slop in, slop out. Nothing beats an organic, pasture-raised, **hand-written** spec. Spec-writing is where the act of *software engineering* really happens.

So a few weeks ago, I started asking myself, how far can I take this? How far *should* I take this?

## Dreaming in markdown

As the story goes, I fell into an AI psychosis, I became a "spec maxxi", and I spent hours and hours writing the most beautiful PRDs and TRDs you've ever seen.

I drafted templates and skills and roles, thinking that maybe my agents can write specs too! I assembled an army, working together like a mini *dark factory,* to turn my specs into reality. My tasks grew more ambitious, and at one point I broke the vibe-coding sound barrier: *an agent that ran for 1.5 hours unsupervised!*

Exciting. But what did that army ship for me? Well, it wasn't slop, in fact it *worked*, which is more than I can say about the garbage that other companies force me to use every day.

But it was still a bit sloppy. I'm far from a perfectionist and I love cutting corners more than most, but this somehow wasn't good enough.

One hallmark symptom of AI psychosis is *using AI to build AI harnesses for building products*, rather than just using AI to build the damn product. I embraced my illness, threw out the branch, scrapped all my markdown, and started all over again.

## Acceptance Criteria for AI (ACAI)

A few days later, I noticed an ambitious little sub-agent doing something unexpected.

```
# Requirements

AUTH-1: Accepts `Authorization: Bearer <token>` header
AUTH-2: Tokens are user-scoped, providing access to any of the user's resources
AUTH-3: Rejects with 401 Unauthorized
```

```javascript
// AUTH-1
const authHeader = req.headers["authorization"];
// AUTH-2
const isAuthorized = verifyBearerToken(authHeader);
// AUTH-3
if (!isValid) return res.status(401).json({ error: "Unauthorized" });
```

**The little guy just went and numbered my requirements and then referenced them all over my codebase.**

Why? I did not ask for this! I was disgusted. This is a tight coupling of code to spec, and spec to code, which is bad right?

*You really expect me to refactor all my code every time I change my spec?*

Oh. I suppose that's a good thing? Interesting. I wonder...

* Perhaps these tags can help me navigate these massive PRs?
* Perhaps they can point me to **where, exactly, a requirement is satisfied or tested!**
* Perhaps I can annotate them with **notes and states (todo, assigned, completed)!**
* Perhaps I can start tracking **acceptance coverage** instead of test coverage!

I leaned in. I named these tags **ACIDs (Acceptance Criteria IDs).**

But a few questions remained.

* *Can my ACIDs number and label themselves?*
* *Is it cumbersome to keep them aligned?*
* *How do I share specs and progress across sandboxes, branches, features and implementations?*

## Acai.sh - an open-source toolkit

I built Acai.sh to solve some of these newly invented problems. And I'm very excited about the results.

* A simple and flexible template for feature specs, called `feature.yaml`. Feature.yaml makes it possible to reference each requirement by ACID.
* Tiny CLI to power your CI and your agent (available on npm or via github release).
* Webapp that serves a dashboard, and a JSON REST API (Elixir, Phoenix, Postgres).

I will keep the hosted version **free for a while, or maybe forever** depending on how popular or expensive this gets. The source code is on GitHub under an Apache 2.0 license.

## How it works

### Step 1 - Specify

Start by writing a spec for a feature.

Be ambitious-- something that adds real value. Don't put nitpicky UI and nail polish stuff in your specs. Keep the requirements concrete, testable, and focused on what really matters (functional behavior + critical constraints).

Rather than markdown, use acai's `feature.yaml` format instead. A spec in Acai is just a numbered list of requirements.

```yaml
feature:
    name: imaginary-api-endpoint
    product: api
    description: This is an example feature spec for an imaginary REST API endpoint, using the feature.yaml format

components:
    AUTH:
      name: Authn and Authz
      requirements: 
        1: Accepts Authorization header with `Bearer <token>`
        1-1: Token must be non-expired, non-revoked
        2: Respects the scopes configured for the owner
        2-note: See `access-tokens.SCOPES.1` for complete list of supported scopes

constraints:
    ENG:
      description: Constraints are for cross-cutting or under-the-hood requirements. Here are some example engineering constraints;
      requirements:
        1: All actions are idempotent
        2: All HTTP 2xx JSON responses wrap their payload in a root `data` key
```

The key benefit of this YAML format, aside from parsing support, is that that each requirement can still be referenced by it's unique and stable ID, e.g. `my-feature.ENG.2`.

### Step 2 - Ship

Copy and paste the prompt below:

> Dear Claude,
> I hope this email finds you well.
> I am writing to ask if you could please do another task for me.
> Start by running `npx @acai.sh/cli skill`.
> This will teach you everything you need to know about our process for spec-driven development. Then, proceed to plan and implement the features specified in our spec files.
> Love,
> [your-name]

### Step 3 - Review

No more file-by-file GitHub PR reviews. Use the Acai.sh dashboard to review *requirements* instead.

Ideally, you just add `acai push` to a GitHub action.

1. Create a free Team and Access Token at https://app.acai.sh
2. Expose the environment variable
3. Push specs and code refs to the dashboard for review

You (human or robot) can mark requirements as Completed when they are ready for review, Accepted when they pass your review, or Rejected if something was missed.

### Step 4 - Iterate / repeat

The goal is to work spec-first.

Change the spec itself, or use the dashboard to attach notes to the spec. With the right harness, you can get your agents to react and self-assign (using `acai` CLI commands).

## Future Gazing

Even coming out my LLM psychosis, I'm still finding this approach useful and productive. I believe I've found the sweet spot between rigour and vibes, structure and flexibility.

### Thought Experiment

Imagine your entire application, however complex, was generated instantly the moment your fingers started typing in the prompt window. Imagine that magically, the same prompt input always created same deterministic output, and cost you nothing, and was ready in milliseconds.

If you found the output to be unsatisfactory (incomplete, insecure, unfit for purpose), you wouldn't start hand-editing files yourself would you? You would extend the prompt to improve it. So if software were free and infinite and instant, your criteria for acceptability is really the only thing of value. The spec.

In the past we spent our time writing down procedures (as code), or writing down invariants (as unit tests), and more recently we've been writing out deltas or changesets to evolve our software (as prompts). It's hard to admit that these are all becoming largely disposable, or invisible, when they used to be the primary object of our attention.

My point is, the spec must live *somewhere*, even if you don't write it down. The spec is what you *want* the software to be. It often exists only in your head or in conversations. You and your team and your business **will always care what the spec says**, and that's never going to change. So you're better off writing it down now! And I think that a plain old list of acceptance criteria is a good place to start.

### From Specsmaxxing to Testmaxxing

When code is generated faster than you can read it, the bottleneck moves to QA and validation and your *confidence* that the code is to-spec. So investing heavily in QA automation and test coverage has a massive ROI.

### From Testmaxxing to reactive software factories

The final unlock is when your LLMs can react to a red test or alert. If the specs are well defined, and you have high confidence in your integration pipelines and QA, then the LLM is fully empowered to react without the need for you to intervene.

## Comparison to other spec-driven development tools

### GitHub SpecKit

SpecKit reads to me like 'vibe coding with extra steps', i.e. a CLI that augments your agents with more prompts and more skills. Solving a slightly different set of problems.

### OpenSpec

The first thing I noticed when reading their docs, is their stated mental model: "Specs [...] describe how your system currently behaves."

I fundamentally disagree with this. In Acai.sh, specs describe how your system **SHOULD** behave. Current behavior is transient.

In addition, their process appears to lean into "AI generated spec writing", which has never gone well for me.

Lastly, the process of versioning and diffing specs is just basic gitops.

### Kiro

I don't like EARS syntax, but I don't like unstructured markdown either, and Kiro claims to convert the latter into the former. Acai is different-- feature.yaml strikes a balance between unstructured markdown and cumbersome EARS / gherkin. Unlike Kiro, Acai.sh does not try to solve end-to-end delivery either (yet).

### Traycer.ai

Looks like a very useful product for managing artifacts and spawning agents to implement them. But they are still plain-old .md files. Acai is more opinionated about how you should write the spec (feature.yaml).

## Reasons you might not like acai.sh

* You might not need to write specs. If your product is low-stakes, or simple, just keep prompting.
* Acai specs are opinionated. You should write 1 spec per feature.
* ACIDs rely on stable numbering. This creates zero friction when drafting a new spec, but requires care when revising a spec: you must re-align the code whenever your spec changes.
* Acai discourages you from putting design and superficial requirements in your specs. Specs are for behavior, and constraints, and nothing else.
* Acai requires you to adopt the feature.yaml format.
