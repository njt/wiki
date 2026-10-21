---
url: https://don-smith.github.io/archie/
date_fetched: 2026-10-21
---

Review

### A finding is not a pass

Architecture Review returns advice you triage. Accepted phases become proposed work items. No set of findings adds up to an architecture approval.

An architecture agent for a code repository

Archie assesses a system's architecture, reviews one module in depth, maintains documentation that cites the code it describes, and checks dependency rules deterministically. It installs into your repository as a set of agent skills plus a pinned runtime, and it works inside whichever coding agent you already use.

01

Capabilities

Archie reads the repository, works out which capability owns your request, and says so before it starts. Nothing is bundled into one universal architecture verdict.

| Capability | Use it when | What you get | 
|---|---|---|
| Architecture Assessment | A whole system needs a recovered fact model before redesign, alignment, or a large refactor. | A validated fact model, current and intended views, explicit gaps, and evidence-led findings. | 
| Architecture Review | No work item is open, but one bounded module deserves a layer-by-layer structural review. | Triaged findings and a phased polish plan in one living artifact. | 
| Architecture Docs | Architecture pages, their claims, and the evidence behind them need writing or refreshing. | A claims ledger, authored pages in a fixed order, a preview, and a handoff for the site. | 
| LikeC4 authoring | A C4 model or a view has to answer one named architecture question. | A compiled model, the selected view IDs, source links, assumptions, and gaps. | 
| Conformance onboarding | A TypeScript repository needs its current dependency structure observed before anyone writes rules. | An observed import graph, an onboarding summary, and a setup proposal to review. | 
| Architecture Contracts | A maintainer is ready to state intended dependencies and have them checked on every run. | A precise contract or exception, and a deterministic conformance result traceable to its rule. | 
| HTML Design | Architecture content needs a readable, accessible, self-contained presentation. | A checked HTML site, review packet, or standalone document. This page is one. | 

02

Authority

Architecture decisions stay with the people who maintain the repository. Archie's job is to make the evidence and the consequences visible first.

Archie labels what it found as fact, inference, or unknown, and it names the capability that owns the next step. When a capability or one of its dependencies is unavailable, Archie reports that gap rather than simulating the workflow and handing you a result that nothing produced.

Each capability keeps the meaning of its own result. That matters most when results start to look like approvals:

Review

Architecture Review returns advice you triage. Accepted phases become proposed work items. No set of findings adds up to an architecture approval.

Conformance

The import graph records what the analyzer saw under the roots you approved. Observed code never promotes itself into an active rule; a maintainer writes the rule.

Model

A LikeC4 view that compiles has valid syntax and resolvable references. Approved claims and their evidence are what make it true.

03

Install

Archie is not published to npm. You install it from a clone of the repository, which is all you need: no registry account and no token.

            There is no `curl | bash` one-liner, deliberately. Cloning first means you
            can read `install.sh` before you run it, and it is commented throughout for
            exactly that. The install needs Node 24, npm, git, and APM on your PATH, and it checks
            all four before it starts.
          

            What it leaves behind is a pin: the exact skills, the runtime tarballs, and a release
            record with a hash for every file. A fresh branch or worktree carries the pin without
            the installed tree and hydrates from your local npm cache the first time Archie runs.
            The install never pushes, never tags, and never touches your application's
            `package.json`.
          

                  Invoke the `archie` skill in your coding agent and describe the
                  architecture question. Archie routes from there.
                

`.agents/skills/`
                  The root skill plus one per capability, deployed by APM and shared with your team.
                  Claude Code users also get a symlink per skill in `.claude/skills/`.
                

`.archie/runtime/`The analyzer, the documentation commands, and the conformance CLI, pinned by lockfile and by hash. Archie brings its own tools rather than asking your repository to depend on them.

`.archie/`
                  Assessments, reviews, and contracts live in the repository beside the code
                  they describe. Uninstalling replays the install journal to undo exactly what
                  the install did, and it keeps `.archie/assessments/` rather than
                  deleting your work.
                

04

Status

What is true today, and what is still moving.

            Archie is at version `0.3.0`. It is verified by installing it into
            real repositories and using it there, not by a formal review gate, and it is developed
            and used on macOS and Linux. The capability boundaries on this page are the ones the
            product actually enforces; the polish around them is still moving.
          

Archie is MIT licensed and developed in the open. Its own documentation, its product boundaries, and its roadmap are all in the repository.
