---
url: https://typesanitizer.com/blog/code-review.html
date_fetched: 2026-08-14
---

# Reviewing code is a skill

**Intended audience:** Software developers interested in getting better at developing software. Initially, I meant this post more for junior developers, but some parts are more applicable for tech leads etc. So if the post appears a bit muddled, please accept this apology from my inner Pascal, “Sorry, I did not have the time to break it up into two posts.”

In developer communities, there has been a lot of discussion
on code review, especially over the course of 2025-2026.
For example, you may have seen claims like:Not necessarily all from the same people.



- “Code review is the bottleneck”
- “Mandatory pre-merge code review is for low-trust environments; you should push directly to main”
- “Code review doesn’t find bugs”
- “Code review is not for X, it’s for Y”
- “LLMs are better than humans at reviewing code”
- “LLM code reviews are much better than humans at finding edge cases/bugs”
- “You should stop looking at the code; you should be doing XYZ instead”

and so on.

Out of this, zooming in on “Code review is not for X, it’s for Y” for a moment, the research has the following to say:

By coding our interview data, we identified four key themes for what Google developers expect from code reviews: education, maintaining norms, gatekeeping, and accident prevention. Education regards either teaching or learning from a code review and is in line with the initial reasons for introducing code review; norms refer to an organization preference for a discretionary choice (e.g., formatting or API usage patterns); gatekeeping concerns the establishment and maintenance of boundaries around source code, design choices or another artifact; and accidents refer to the introduction of bugs, defects or other quality related issues.


Similarly, Expectations, outcomes, and challenges of modern code review (2013) points out:

Our study reveals that while finding defects remains the main motivation for review, reviews are less about defects than expected and instead provide additional benefits such as knowledge transfer, increased team awareness, and creation of alternative solutions to problems. Moreover, we find that code and change understanding is the key aspect of code reviewing and that developers employ a wide range of mechanisms to meet their understanding needs, most of which are not met by current tools


So at least, hopefully we can agree that code review serves many purposes.

I will get to the other points in a bit.

But before that, I want to articulate a framing which I haven’t seen much elsewhere.
Namely, that *reviewing code is a skill*. Specifically, I posit that:

- It’s possible to get better at reviewing code. By “better”, I mean across all of the purposes above: catching bugs, catching design issues, increasing one’s awareness of what’s going on, and understanding the code.
- It’s possible to teach someone to get better at reviewing code.
- By virtue of it being a fairly modern skill, we don’t quite know where the human skill ceiling lies (e.g. what is the Pareto frontier in terms of speed vs quality?).
- If you’re a software developer, and if you believe that people
will continue to be involved in the development and maintenance
of programs for the foreseeable future, then it’s *valuable*to get better at reviewing code.

First, I will give three small examples taken from the past few weeks of work of me finding bugs when reviewing code. I’ve specifically chosen bugs for discussion because they’re relatively unambiguous.

Next, I will provide some of my own historical background related to code review, and some arguments in favor of the core thesis.

After that, I discuss some ideas for experimenting with and improving code review.

Finally, I discuss the aforementioned oft-repeated memes around code review and how they hold up to scrutiny with the thesis in mind.

Let’s get started.

## A tale of three almost-introduced bugs

A bunch of details have been elided in the course of explaining the different examples below, in an attempt to make them digestible. It may be valuable to keep an eye on stray thoughts of the form, “well, that seems like a code smell, no wonder you almost had that bug” or “duh, this could’ve been avoided with XYZ”.

In two out of the three cases, the person who wrote the PR had experience with the surrounding code.

The other thing to note is that LLM reviews with a mixture of high-end coding models (around Jun 2026) were run for all of the PRs described below. They did not catch the issues that I caught.

It may be helpful
to consult this handy table from Lorin Hochstein’s
Traditional versus resilience engineering views:The post is short and recommended reading. The table below is a subset of the one in the original post.



| Traditional view focuses on | Resilience engineering view focuses on | 
|---|---|
| objectives | production pressure | 
| reducing complexity | navigating complexity | 
| root cause | interaction of multiple factors | 
| human variability as liability | human variability as asset | 

One way to read the cases below is to try to think about both sides of the table as you read them.

### Writing some git configuration

We use our own devboxesDisposable VMs used for software development.


 at `$WORK` which run on EC2 instances.
The logic for starting things has two subprocesses:

- One background process, to initialize state which isn’t needed right away. This process may or may not have finished by the time the user starts using the devbox. 
- One foreground process, which needs some extra data from the user’s laptop, and blocks the user until it’s done. Only after this process finishes can the user start using the devbox. 

To reduce latency, we’ve been working on
moving more operations to the background process.
In this spirit,
one of my colleagues created a PR to move some of
the modifications to the global `~/.gitconfig`In the interest of performance and consistency, we want to manage some aspects of people’s Git configurations. Yes, I know Nix exists, I use it for one of my own servers. No, we don’t use Nix at work. You’re keeping a check on your intrusive thoughts, right?


from the foreground process to the background process.

So when a `git config` command needs to modify `~/.gitconfig`,
it first acquires an exclusive file lock on `~/.gitconfig.lock`.
This prevents concurrent modifications from other (cooperating)
processes, such as other `git config` calls.

When I saw the PR, I remembered we’d hit non-determinism in the devbox setup, where git’s behavior of fast-failure on lock acquisition failure led to flakiness during startup due to a similar problem of concurrent writes.

Just retrying on top would’ve still led to non-determinism, so my colleague added a cross-process dependency edge using an existing mechanism, where the write would only happen after the background process finished.

Then I pointed out that we’d actually tried that earlier,
but we’d gotten rid of that almost immediately,
due to the increase in end-to-end latency
(because now a subpart of the foreground process
would have to wait for the *whole* background process
to finish).

In the end, since we also had some file modifications which
couldn’t quite go through `git config` directly,Due to needing to reconcile `# DO NOT EDIT` blocks.


we converged on a solution which had a separate `flock` operation
(with its own `.lock` file). This allowed for (1) having retries
with backoff (2) doing multiple modifications under the same `flock`
without other intervening writes
(3) doing direct writes without worrying about concurrent writers.

### To display progress or not to display progress

There’s a periodic CI job which does some processing
and uploads a tarball to an AWS S3 bucket.
Turns out, the `aws` CLI displays progress by default.
Presumably, this is meant to help with debugging issues,
as well as provide reassurance about progress
when using the CLI directly in the terminal.

When this colleague changed the logic to upload to
4 buckets from 1 bucket to speed up downloads from other
regions,AWS buckets belong to a specific region.


 the CI job started failing,
due to the log file exceeding the limit of 10MB.

Turns out, the `aws` CLI logs a line for every 256 KB uploaded.
This corresponds to tens of thousands of log lines when the upload is 10GB+.
Multiplying the log lines by 4 tipped the job over the 10MB limit.

The colleague submitted a PR to change the invocations to
use `--no-progress` to unbork the CI job.
At the time, my first thought was, “Hmm, is this the only option?
Maybe there’s a way to show fewer progress updates?
If the job fails mid-upload, it’d be nice to have some
progress information to make it clearer *when* the job
failed relative to the start of the upload.”

So I looked up the docs for the `aws` CLI, and saw a flag
`--progress-seconds <INT>` which allowed tuning the frequency
of progress updates. I asked the PR author if we could use
that instead.

At the back of my mind, I remembered a mini-incident
we’d had earlier, where a commonly used script had introduced
an `aws` CLI flag, but not all environments the script ran in
had a sufficiently new CLI version which supported it,
so the script broke for a bunch of people.

Since the PR author had also seen that happen,
I assumed that they would do the due diligence to check
what version of the CLI was running in the CI job and
check whether that CLI version supported `--progress-seconds`.

The PR author got back to me quickly and updated the PR to
remove `--no-progress` and use `--progress-seconds`.

I was a bit surprised at the speed, so I realized they might
not have thought of the same risk. First, I tried looking
at the aws CLI changelog to see if it mentioned when the flag
was introduced (since the flag’s documentation didn’t have this
information). It did not. Then I had LLM chase down
what CLI version the CI job was using,
as well as which commit introduced that flag in the `aws` CLI,
and the first published release with that commit.
This uncovered the fact that the job would break
if `--progress-seconds` was used,
because the existing CLI version in the job was too old.

I did some spot checks on the commits and versions, and then commented on the PR with a brief apology for not stating my assumptions earlier, and asked if they could cross-check the findings/find an alternate solution.

The PR was later fixed by installing a sufficiently new
`aws` CLI version that was already being packaged
in another place in the repo.

### The extra SHA that could’ve caused an outage

Remember the CI job from the previous example?
One of the things I’ve been working on lately is decoupling
the release process of that job group from the overall
CI release process.The CI system is complicated enough
that different jobs can have their own release processes.



One day, I hit a merge conflict while rebasing my WIP
changes on top of the `master` branch. Turns out,
someone had added new variations of the existing job.

So the job group is responsible for uploading certain tarballs. An engineer needed some more tarballs to be uploaded slightly differently. They’d created a separate job for that, instead of updating an existing job in the group.

I was like, “hmm, why is this a different job, shouldn’t it involve adding one more element to a list somewhere?”

Turns out, the reviewer for that job-addition PR had
recommended separating out the new job from the old one
to start with to reduce risk with disrupting the existing job’s
performance, reliability etc. Additionally, one of
the “slight” differences was that the new job was uploading
sidecar files with a checksum.If you’re reading this and going “but doesn’t S3 already natively support checksums”, I actually didn’t know about that until after all of this was done-and-dusted, and I’m guessing the original PR author didn’t either.



I was confused looking at that. “If this job needs checksums, do the existing jobs need checksums? If so, why didn’t they make the checksum logic uniformly applicable to all jobs? Do we need to update the existing readers to check the integrity of the old tarballs? Are these new tarballs crossing some kind of trust boundary whereas the old tarballs were not?”

Thinking a bit more, I realized there was a bug. About a year back, I’d watched a presentation by Prof. Hannes Mühleisen called DuckLake - The SQL-Powered Lakehouse Format for the Rest of Us. It was talking about Apache Iceberg and its use of files, which seemed very complicated.

At the time, my understanding was that some of this complexity was due to essentially not having a way to do multi-object transactions. And thus, one of the key design decisions for DuckLake was to have a SQL database (with ACID transactions) for maintaining metadata.

Coming back to the CI job, it was uploading the tarball
to a fixed object name in a fixed bucket. This was uploaded
before the checksum. So while S3 guarantees all-or-nothing
semantics for a write to an object,
if the job was canceled or crashed before uploading the checksum,
then readers checking the integrity of the tarball
against the old checksum in a fail-closed mannerThe usual way to use checksums.


would start failing, which would lead to an ~outage
for that particular feature until the tarball
and checksums were back in sync.

I pointed out this failure mode to the person who created the PR, as well as in the PR introducing the read path. In the end, the read path check for the sidecar file was not introduced.

## The case for code review being a skill

Let’s rewind the clock back a bit. To 2018.

I was working as a graduate student in physics. The research I was doing involved various kinds of simulations. We used Jupyter notebooks for much of the work. The simulations would work one day, they’d stop working the next day, and the third day, they’d continue not working despite undoing the work from the second day.

I was struggling with how to manage the code, how to make it efficient, and how to prevent mistakes from coming up, and how to use version control. At the time, I started learning about automated testing and code review. I recall being quite amazed, “Wait, what? People actually review all of the code, not just the outputs?! Wow, it must be nice to work as a software developer.”

Eventually, I got increasingly more interested in the process of writing software better, and more disenchanted with the physics I was supposed to be studying. I ended up dropping out of the PhD program in 2019, and started working as a software engineer.

Since then until now, over the course of my career so far, there have been multiple times where someone have commented that they’ve found my code reviews more helpful than the average review they’re used to.

As one data point, before 2026, when reviewing a PR, depending on the author’s familiarity with how I review, the underlying code, prior discussions etc., my comment density on PRs has been in the range of 30-100 SLOC/comment. Not all of them were about bugs. Most were about more “boring” things like asking clarifying questions, naming, layering etc.

So why am I telling you all of this? Well, my point is that I learnt how to review code along the way. I was not magically born with some innate ability to do code review, or to catch bugs in code review.

Are there aspects somewhat specific to me which help me in spotting bugs? Probably, yes. Some things I can think of:

- When I discover a bug that I introduced earlier, I tend to take it more personally than other developers I’ve met.
- I tend to think of programs in terms of invariants and little proofs.
- I like reading tech blogs and watching tech talks, especially those with debugging stories and performance investigations.

Out of these, I think the second point in particular is very learnable and teachable.

Stepping back a bit, I think there’s a *lot* we don’t
really know about code review. If you look at the literature
on code reviews, it’s generally pretty anemic.

Apart from doing more structured studies – which would be difficult to get buy-in for in a typical corporate environment – I think there’s a lot of room for trying out new practices related to code review, including at small scales, to make it more useful for authors, reviewers and the surrounding team.

## Putting on the mad scientist hat for a bit

In this section, I want to suggest some potential ideas to improve upon (or at least, try variations of) code review as a practice. The purpose of writing these down is twofold:

- To give you some inspiration for coming up with ideas of your own.
- To give you some sense of how existing practices are lacking in particular aspects

The point is not “you should go try these out” (maybe you shouldn’t) or “I am certain that these will work well” (I’m not, because I haven’t had the chance to put them into practice).

The ideas below assume a strong foundation of psychological safety, because if you don’t have that, you should almost certainly be fixing that first before trying out more weird things.

### Randomized process-oriented Socratic dialogues

Say you have a junior engineer who is requesting a review from a senior engineer with expertise relevant to the code being changed.

When the PR is up, and the review is requested, say a bot randomly decides (based on some tunable frequency) to create a meeting.

In the meeting, the senior person, instead of pointing
out things which don’t quite make sense,
or could be done differently, instead
first asks the junior person for their perspective
on why they did things a certain way,
or what assumptions they were making.It’s key to avoid counterfactual questions for why something was *not* done a certain way, or why the person did *not* think of something. This might sound easy, but if you actually try this, you’ll notice it’s quite hard to stick to this! (EDIT: If you’re wondering “why should I avoid counterfactuals”, you might find my previous post How should we learn from my bugs? and Lorin Hochstein’s The Problem with Counterfactuals useful.)



During this process, a junior with cognitive agility will recognize gaps in their understanding, and potentially identify improvements they could make.

The focus would be on the *thought process* for
different sub-skills like writing a good PR description
or robust error handling, with the PR serving
as a foil to ground the discussion.

See also: 6 principles of Effective feedback, §1. Effective feedback is based on process, not outcome.

### Lightweight near-miss post-mortems

Today, it’s increasingly more common to have post-mortems
for incidents. It’s also common to organize work into
sprints and have team retro meetings at the end
of a sprint.Also a skill that can be learned, improved upon etc.!



What if, every time a bug was caught in code review during
the sprint, the PR author was asked to record a short clip
explaining the context and what the caught issue was,
and if the kind of issue matched a past issue,
what the impact of the past issue was.The wording here is very deliberate. I’m explicitly not mentioning “what the potential impact could’ve been had the issue slipped through” because that goes into the territory of speculation/prediction.



During the team retro, the team could watch the clips together, ask questions and increase shared knowledge. This way, everyone on the team would also be exposed to a lot more bugs that almost slipped through.

To avoid having differences in frequency, you could limit this to at most one clip per person per meeting.

See also: You’re missing your near misses.

### Firewalled modeling

In Finding bugs without running or even looking at code (video) (StrangeLoop 2019), Jay Parlar describes an example of him trying to build a model in Alloy for an access control system – which spanned multiple codebases, and where he hadn’t looked at the code – and in the process of modeling, he found both bugs and design issues.

In my own limited experience, lightweight formal methods seem useful when trying to reason about process semantics (involving cancelation, potentially indefinite waits, file locking protocols) as well as access control.

What if, as a baseline, when you started working on a system,
especially a complex one which interfaced with multiple
external systems or carried high security or correctness
risks, two people played the roles of the modeler
and the programmer.
The modeler would develop the model *without* looking at the code,
and the programmer would work on the code.
They’d meet in the middle to create test cases.

Then the model could serve as a much smaller reference to be able to review the thoroughness of the code, as well as to test out various unusual situations that would otherwise require a large amount of test scaffolding, especially in the early stages of the project.

### Studying expertise

(OK, this one doesn’t fit under “mad scientist” but whatever. It’s my blog.)

In many fields, there is a growing body of literature around expertise. This doesn’t really exist for code (and specifically code review) in quite the same way.

What if you analyzed the code reviews happening in your team and tried to figure out if there are outlier (domain, person) pairs in terms of insightful review comments. Or maybe you get this information just from interviewing people.

Once you’ve found the outliers, you could try an easy-to-learn approach like Applied Cognitive Task Analysis for extracting the tacit knowledge these people have. Doing this is easier for code review compared to many other tasks, because instead of simulating a difficult-to-simulate situation (such as firefighting), you can actually just have the person review PRs live.

## On the zeitgeist surrounding code review

Hopefully, by this point, I’ve convinced you somewhat that reviewing code is a skill, and that there’s a lot more room left for us to try out different practices. For example, in terms of training programs, we are not close to more well-studied areas like team sports (e.g. soccer) and two-player games (like Chess, Go etc.).

At this point, you might (reasonably) still be thinking:
“So what? LLMs are going to get better at all of these things
much faster than humans can get better at them,
so it’s pointless to try to compete with LLMs
at reviewing code.”A related objection might be that while you might care about code review, your management doesn’t. Sorry, I don’t have any advice related to convincing management.



If you’ll recall my phrasing from before:

If you’re a software developer, and if you believe that people will continue to be involved in the development and maintenance of programs for the foreseeable future, then it’s

valuableto get better at reviewing code.

The second part of this is conditional on the first. So we can disagree on what likelihood we assign to the premise itself (“if you believe … for the foreseeable future”) being true, but that’s a separate discussion from whether the implication (the “then”) itself holds.

The flipped version of the implication would be something like:

If you’re a software developer, and even if you believe that people will continue to be involved in the development and maintenance of programs for the foreseeable future, it’s

net negativeto get better at reviewing code.

For example, you could hold this position because you think
that there are other skills which you could learn that provide
significantly higher leverage.I commonly see “system design” and “product thinking” or “developing taste” as being touted as more important to develop.


Or you might think that reviewing code will later
be restricted to a small fraction of working programmers,
like how a relatively small fraction of programmers today
look at assembly code on a regular basis.

To this, I have the following responses.

First, if you’re able to drop down a lower-level of abstraction compared to your peers, that’s almost always an advantage because you’re able to tackle a wider set of problems that they can. For example, today, if you’re able to understand SQL query plans more easily, or you understand things like memory allocation and assembly, you are at an advantage at designing more robust and high-performance code compared to engineers who don’t understand those topics.

Second, given that software development is still in its infancy compared to so many other fields, it’s quite possible that we’re not even close to seeing the ceiling on human skill and performance on various aspects, such as, but not limited to, code review. So it may very well be that the average skill you see around you may be far from the ceiling is.

Lastly, I recommend grounding your world view in experience reports, case studies, and your own observations, rather than “takes” on social media. Cedric Chin has written about much better about this than I could’ve in How to make sense of AI and Letter to a young person worrying about AI.

At the end of the day, if you’re going to be responsible for the code you ship, and the code you ship has real consequences for real people, it’s worth asking how you can get better at what you do. To that end, I believe that investing in getting better at reviewing code is one of the best things you can do as a software developer.

Yaksha: What is swifter than the wind?

Yudhishtira: The mind.

Yaksha: More numerous than the blades of grass?

Yudhishtira: The thoughts in the mind.

Yaksha: What is the most praiseworthy thing of all?

Yudhishtira: Skill.

Yaksha: The most valuable possession?

Yudhishtira: Knowledge.
