---
url: https://notes.ito.com/53165f08d3f68ceafe9a5ab9/
date_fetched: 2026-09-15
---

Note for Harper

# How I run my agents across many accounts

You noticed I spread usage across a bunch of accounts. The accounts are the bottom layer of a system I call Mujin. Here is the whole thing.

Most of what I do is write issues: ideas, bugs, things I want built. Agents groom them, build them, review them, land them, and review them again after landing. I named the whole pipeline Mujin, 無人, after the Japanese term for a lights-out factory. Attended sessions still produce work. Mujin is what makes them optional.

The organizing principle is that no producer lands its own code. Main has exactly one writer, and a bounce is the normal retry path. The safety principle is that a human is the only path to the label that makes an issue executable, and the grooming lane that proposes that label cannot write it.

## 1. The tracker and the gardener

Everything starts as a kata issue. kata is the issue tracker Kenn built; I use it for ideas, bugs, ops signals, and decisions I owe someone. My agents comment on issues, claim them when they start, and release them when they stop. The owner field means "in flight right now" and nothing else, so an idle claim is a signal that something died.

Niwashi (庭師, gardener) is the grooming lane. It runs on a schedule, one project at a time, in windows across my afternoon and evening. Each run takes up to six unowned issues and a read-only checkout of the repo at a pinned commit, and runs two model stages: a proposer writes a short implementation card or a hygiene proposal, then a reviewer from a different model family checks every file and line claim against the pinned commit. The two stages run as a credential-less OS user. The run is allowed to write a comment and a candidate label, and nothing else. It cannot close, move, reprioritize, or edit an issue.

The label that makes an issue executable by the night lane is designed-ready. It has exactly two writers: me, through a promotion command under my own identity, and an auto-promoter that only takes low-risk cards the reviewer passed, at most three per run. Everything with a real tradeoff comes to me as one Telegram message per decision with an approve and a skip button. After I once approved a card and eight decisions without understanding any of them, every message now has to open with six fixed lines: the ask as a question, what happens if yes, what happens if no, a recommendation, why this needs me specifically, and how to undo it. A validator refuses messages that break the shape.

Niwashi also derives a route for each issue from two independent model readings, one for sensitivity and one for the kind of coding it needs. Public and bounded goes to GLM, internal and bounded goes to Codex, UI and vision drafts go to Kimi, and anything sensitive or ops-shaped stays with an attended Claude session. The route is a recommendation until a human promotes the issue.

## 2. Producers

Three kinds of producer build branches, in parallel, each in its own git worktree.

**Attended sessions.** Me plus Claude Code or Codex. For anything non-trivial I run /do-it, an autonomous pipeline: spec, plan, build, review, commit, push, with questions allowed only before the spec. The process layer is superpowers, Jesse Vincent's skills library, which decides how an agent brainstorms, writes a plan, tests first, debugs, and reviews. Execution of the plan goes through Agency, which evaluates each task before moving on. A fresh subagent picks the starting review rung from a rubric, because the executor's incentive is always to go light. Light is one same-model review pass, medium is one independent-model pass through fresheyes, heavy is a loop with a diminishing-returns judge capped at six passes or ninety minutes. The rung only ever climbs.

**Dispatched workers.** When I want an issue built now, I run kata-dispatch against it. It refuses unless the issue is open and unowned, claims it, creates a worktree, copies the brief into the worktree, splits a tmux pane, launches the harness, sends the kickoff, and verifies the kickoff was accepted by watching for the TUI's working state rather than the echoed prompt. Then it arms a per-dispatch liveness watcher and installs a hook that writes the session's state to a file. Every dispatch gets a model, seat, and worktree recorded in the issue's metadata.

**The unattended night lane.** Between 22:00 and 06:30 my local time, one bounded pass selects unowned designed-ready issues, up to five per night, and runs at most two headless Claude sessions at a time on a pinned seat. Each issue gets ninety minutes, with a soft warning at seventy-five and a stall kill after thirty minutes of flat transcript. The night stops after two failures. Each session ends one of three ways: it submits to Repoman, it fails and leaves its worktree as forensics, or it escalates by pushing its work in progress, writing the decision it needs as a comment, labeling the issue for my decision, and stopping. Nobody is awake for any of this.

## 3. The seats, and how a dispatch picks one

This is the part you noticed. I hold multiple Claude Max and Codex Pro seats, personal subscriptions, plus my ordinary logins, which also count as seats. Each seat has its own profile directory and its own credentials. The pool tracks each seat's five-hour and weekly usage from Anthropic's own meter, polled every five minutes, with a transcript-based estimate as a fallback because headless sessions never report a status line.

A dispatch reads the issue's model labels first. model:fable, model:astra, and model:no-fable each force a lane, and contradictory pairs are refused rather than resolved. An unlabeled dispatch runs a three-way headroom gate. The Fable lane is eligible when some Max seat's Fable share is under forty percent. The Astra lane is eligible when some Codex seat is under sixty percent. Each is measured against its own line because the meters are different units, the lane with more headroom wins, a tie goes to Fable, and neither being eligible means Opus. A meter that is missing, older than two hours, malformed, or dated in the future makes that lane ineligible. Unknown never scores as probably free. I should add Sol as a further fallback.

Within a lane, "coldest" means the worse of the seat's five-hour and weekly meters, plus ten points per live session already on that seat, with a least-recently-used tie-break stamped in microseconds. The live-session term exists because one morning a burst of launches all read the same five-minute-old snapshot and piled a dozen Fable sessions onto one seat while other seats sat at forty percent with nothing on them.

A separate sweeper runs every ten minutes and looks for an interactive session that has actually stopped on a limit banner. It exits the session cleanly, copies the transcript into a cooler seat's profile, relaunches pinned to that seat in the same pane, answers the resume chooser, verifies the seat from the process environment, and sends a continuation nudge. It moves at most two sessions per run, never touches a pane where someone is typing, and leaves a pane alone if the reset is under thirty minutes away. It is not perfect. It cannot see sessions on the second tmux server my worktree tool uses, and it misses some banner shapes until I teach it a new one, so I still move some sessions by hand.

## 4. Gate, lander, patrol

Nothing lands without an independent review. fresheyes, Dan Shapiro's tool, reviews spec, plan and diff with a model from a different family and no shared context. GPT-5.6 Sol is the fleet default and GPT-6 Astra is used for security-shaped changes and the heavier /do-it rungs. Same-model subagent review does not count.

Repoman is the only writer of main on eight repos. A producer pushes its branch and runs repoman-submit, which comments a marker pinning the exact commit and then adds a handoff label. A watcher on one always-on Mac wakes every sixty seconds and makes no network connection unless there is real work. For each handoff it claims, verifies the pinned commit, rebases onto current main, resolves conflicts with a model restricted to the conflicting files or bounces, runs the test battery in a throwaway clone with no credentials, and pushes with a compare-and-swap so a moved main causes a retry rather than a clobber. Consecutive handoffs for one repo land as one rebased train under one gate, falling back to one at a time if the train goes red, so blame stays with the right branch. Red means a bounce back to the producer with the failure named. There is no retry command. Fixing the branch and submitting again is the retry. GitHub rulesets make everyone else read-only on main, including me through the web UI. Median handoff to landing on the busiest repo is about nine minutes.

roborev, Kenn's post-commit review daemon, reviews everything that lands and posts failing verdicts to a chat room. The fix loop is always a session, never a daemon talking to a daemon.

## 5. Kimi, GLM, and LunaRoute

I have started moving public work off the Claude and Codex seats when they are hot. claude-glm is the official Claude Code binary pointed at z.ai's Anthropic-compatible endpoint, pinned to GLM 5.3, under its own profile with no skills, no plugins, no MCP servers, and no access to kata. It refuses to start unless the working directory is a worktree of one of two allowlisted public repos with no credential files in it, the profile's deny rules are intact, the z.ai quota is under ninety percent, fewer than two GLM sessions are running, and the shell is scrubbed so my fleet credentials cannot leak into the model's tool calls. It is attended only. The first GLM task landed on main two days ago.

Kimi K3 runs only through a batch lane on a credential-walled VM, and only for UI drafts, analysis drafts on public material, and anything needing vision. Two UI tasks have gone through and merged. Polish tasks cost about twice what generative ones do in tokens. I deferred the Kimi consumer coding plan because its terms forbid scripted bulk use and there is not much Kimi demand yet.

The rule for both is three gates in order. Sensitivity: Chinese-hosted providers see public, bounded, self-contained material only, nothing from my vault, people data, email, or business terms. Shape: batch with acceptance criteria, no turn-by-turn steering. Role: never long-horizon multi-tool work, never sole code review, never anything in my voice. Their output is never used as fact without verification.

LunaRoute is Eran Sandler's no-log inference gateway and MCP tool server. I have it wired as an MCP server on every Claude seat for web search, document conversion, and image generation, and as a provider extension in Pi on one Mac. I am testing it, not routing coding work through it. The free tier has one inference lane, so parallel calls stall.

## 6. Staying inside the terms

I only ever run the official harnesses, Claude Code and Codex. The pool is a launcher. It picks a seat, puts that seat's credentials into one child process's environment, and execs the vendor's binary. It never listens on a socket, never holds tokens in a resident process, and never exposes a compatible endpoint. That relay shape is what gets accounts terminated in waves, and one shared network with a handful of egress IPs is exactly the one-source-many-tokens signature. The subscriptions are personal, one person paying for each, not phantom seats on an org plan. Anything that answers other people, like my WhatsApp and Discord bots, runs on API keys, because a personal subscription cannot be shared.

## 7. Knowing when an agent is stuck

This is still the hardest part. Agents pause and fail in different ways: a quota banner, a permission prompt, a startup modal, a crashed pane, a turn that ended waiting on a background task, a session that is still writing but has drifted off spec. For months I ran everything in tmux panes and had other agents read the panes. That is now three separate mechanisms.

**Hook state.** Every dispatched Claude session carries a hook on eight lifecycle events that writes one small file: working, idle, needs-input, or ended, with the session id. A rate limit, an auth failure, or a permission prompt becomes needs-input, which means a human must act. The hook never prints, never blocks, and always exits zero.

**A liveness watcher per dispatch.** It lives as long as the pane and reads the hook file first, falling back to transcript growth, never pane activity, because the TUI redraws its clock while idle. On a stall it climbs a ladder: one nudge, then a nudge plus an alert to my ops channel, then a comment and a needs-review label that parks the issue. It never nudges a session whose hook state says working, and it never nudges needs-input. It exists because five dispatched builds once sat idle for eight to sixteen hours after the dispatcher verified the kickoff and forgot the pane.

**Three levers for the coordinator.** A written directive posted as a kata comment, which the worker reads at its own checkpoints and acknowledges by id. A stop label, which parks the worker at its next checkpoint. Direct conversation only when the send path can prove the worker idle. The send path refuses to paste into a working builder. This came out of a coordinator run that sent 287 steering messages into a builder mid-thought and polled 1,629 times, and landed nothing.

The design change underneath all of this is that the session is the identity and the pane is just where it is attached. The dispatch record carries the harness session id, the send path follows a session into a new pane after a resume, the watcher stays armed when a pane disappears but the transcript is still moving, and the board I look at resolves workers by session id and re-attaches a dead row's session under the profile it actually ran on. Dispatch itself stays inside tmux on purpose, because the worktree tool's own session launcher would bypass the seat gate.

## What is not done

The dispatcher that would act on Niwashi's routes is armed nowhere yet; it runs in shadow mode. The sweeper cannot see the second tmux server. There is no Sol fallback below Opus. The Kimi consumer plan is deferred. And the hardest problem, telling a wedged session from a working one cheaply and early, is better than it was and not solved.
